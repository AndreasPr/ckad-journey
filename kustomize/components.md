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
