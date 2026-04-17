## What are Components in Kustomize?

**Components** in Kustomize are reusable building blocks that bundle:
- Resources
- Patches
- Transformers

They allow you to define **optional features** that can be included in multiple overlays **without duplicating configuration**.

---

## Why Use Components?

Components are useful when:
- You have features that are **not required in all environments**
- You want to **reuse configuration across multiple overlays**
- You want to keep overlays clean and modular

---

## Key Idea

- **Base** → Core application (always included)
- **Overlays** → Environment-specific changes
- **Components** → Optional features that can be plugged into overlays

---

## Example Scenario

We have:

### Overlays:
- `dev`
- `premium`
- `self-hosted`

### Components:
- `caching` → Used in `premium` and `self-hosted`
- `external-db` → Used in `dev` and `premium`

---

## Folder Structure

```bash
k8s/
├── base/
│   ├── deployment.yaml
│   └── kustomization.yaml
├── overlays/
│   ├── dev/
│   │   └── kustomization.yaml
│   ├── premium/
│   │   └── kustomization.yaml
│   └── self-hosted/
│       └── kustomization.yaml
├── components/
│   ├── caching/
│   │   └── kustomization.yaml
│   └── external-db/
│       └── kustomization.yaml
````

---

## Base Configuration

### `base/kustomization.yaml`

```yaml
resources:
- deployment.yaml
```

---

## Component: Caching

### `components/caching/kustomization.yaml`

```yaml
apiVersion: kustomize.config.k8s.io/v1alpha1
kind: Component

patches:
- patch: |-
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: my-app
    spec:
      template:
        spec:
          containers:
          - name: app
            env:
            - name: CACHE_ENABLED
              value: "true"
```

---

## Component: External Database

### `components/external-db/kustomization.yaml`

```yaml
apiVersion: kustomize.config.k8s.io/v1alpha1
kind: Component

patches:
- patch: |-
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: my-app
    spec:
      template:
        spec:
          containers:
          - name: app
            env:
            - name: DB_HOST
              value: "external-db-service"
```

---

## Overlay: Dev

Uses:

* Base
* External DB component

### `overlays/dev/kustomization.yaml`

```yaml
resources:
- ../../base

components:
- ../../components/external-db

patches:
- patch: |-
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: my-app
    spec:
      replicas: 1
```

---

## Overlay: Premium

Uses:

* Base
* Caching component
* External DB component

### `overlays/premium/kustomization.yaml`

```yaml
resources:
- ../../base

components:
- ../../components/caching
- ../../components/external-db

patches:
- patch: |-
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: my-app
    spec:
      replicas: 5
```

---

## Overlay: Self-Hosted

Uses:

* Base
* Caching component only

### `overlays/self-hosted/kustomization.yaml`

```yaml
resources:
- ../../base

components:
- ../../components/caching

patches:
- patch: |-
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: my-app
    spec:
      replicas: 2
```

---

## What Happens During Build?

Example:

```bash
kubectl apply -k overlays/premium/
```

Kustomize:

1. Loads base configuration
2. Applies:

   * Caching component
   * External DB component
3. Applies overlay-specific patches (replicas = 5)

Final result:

* App has caching enabled
* Uses external DB
* Runs 5 replicas

---

## Benefits of Components

* Reusable across multiple overlays
* Avoid duplication
* Enable feature-based configuration
* Improve modularity and maintainability
* Clean separation of concerns

---

## Summary

* Components = reusable optional features
* Used alongside base + overlays
* Ideal for feature toggles like:

  * caching
  * external services
  * logging
  * monitoring

They help you build **flexible, composable Kubernetes configurations** at scale.

# Scenarios

## Scenario 1
A new `caching` component needs to be created for the application.

There is already a directory located at:
`project_mercury/components/caching/`


This directory contains the following files:

- `redis-depl.yaml`
- `redis-service.yaml`

Finish setting up this component by creating a `kustomization.yaml` file in the same directory and importing the above Redis configuration files.

## Solution

Create a `kustomization.yaml` file inside the `components/caching/` directory with the following content:
```yaml
apiVersion: kustomize.config.k8s.io/v1alpha1
kind: Component

resources:
  - redis-depl.yaml
  - redis-service.yaml
```

This defines the caching component using Kustomize and includes the required Redis resources for deployment.




## Scenario 2
With the database setup for the `caching` component complete, we now need to update the `api-deployment` so that it can connect to the Redis instance.

Create a **Strategic Merge Patch** to add the following environment variable to the container in the deployment:

- Name: `REDIS_CONNECTION`
- Value: `redis-service`

Note:

- The patch file must be created at:
`project_mercury/components/caching/` with name `api-patch.yaml`

- After creating the patch file, you must also update the `kustomization.yaml` file in the same directory (`components/caching/`) to include this patch under the `patches` field.

This step is essential — without updating `kustomization.yaml`, the patch will not be applied when the component is used in an overlay.

## Solution

Navigate to the `/root/code/project_mercury/` directory.

1. Create the patch file

Location: `components/caching/api-patch.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-deployment
spec:
  template:
    spec:
      containers:
        - name: api
          env:
            - name: REDIS_CONNECTION
              value: redis-service
```

This patch ensures that the `api` container can access Redis using the specified environment variable.

2. Update the component's `kustomization.yaml`

Location: `components/caching/kustomization.yaml`

Ensure that the `kustomization.yaml` file includes the `api-patch.yaml` under the `patches` field

```yaml
apiVersion: kustomize.config.k8s.io/v1alpha1
kind: Component

resources:
  - redis-depl.yaml
  - redis-service.yaml

patches:
  - path: api-patch.yaml
```

Note:
- If you want to apply this component independently (e.g., using `kubectl apply -k components/caching/`), you will need to include `../../base/` in the `resources` field so that Kustomize can locate the original `api-deployment` for patching.

- However, when this component is used within an overlay like `overlays/enterprise/`, the `base` is already included at a higher level. Adding `../../base/` again inside the component will cause a duplicate resource error during `kubectl apply -k overlays/enterprise/`.