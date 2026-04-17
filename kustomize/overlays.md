## What are Overlays in Kustomize?

**Overlays** in Kustomize are used to **customize a base configuration for different environments** (such as dev, staging, production) without modifying the original base manifests.

They allow you to:
- Reuse a common base configuration
- Apply environment-specific changes (replicas, images, labels, etc.)
- Keep your configurations clean, modular, and DRY

---

## Core Concept

Kustomize is built around two main components:

1. **Base**
   - Contains common, reusable Kubernetes manifests
   - Represents the default configuration

2. **Overlay**
   - Modifies the base for a specific environment
   - Applies patches, transformers, or overrides

---

## Folder Structure Example

```bash
k8s/
├── base/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── kustomization.yaml
├── overlays/
│   ├── dev/
│   │   └── kustomization.yaml
│   ├── staging/
│   │   └── kustomization.yaml
│   └── prod/
│       └── kustomization.yaml
````

---

## Base Configuration

### `base/deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 1
  template:
    spec:
      containers:
      - name: app
        image: myapp:latest
```

### `base/kustomization.yaml`

```yaml
resources:
- deployment.yaml
- service.yaml
```

---

## Overlay Example (Production)

### `overlays/prod/kustomization.yaml`

```yaml
resources:
- ../../base

patches:
- patch: |-
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: my-app
    spec:
      replicas: 5

images:
- name: myapp
  newTag: prod

commonLabels:
  env: production
```

---

## What Happens During Build?

When you run:

```bash
kubectl apply -k overlays/prod/
```

Kustomize:

1. Loads the base configuration
2. Applies overlay customizations:

   * Updates replicas to 5
   * Changes image tag to `prod`
   * Adds label `env=production`
3. Outputs the final combined manifest

---

## Typical Overlay Use Cases

| Environment | Customizations                          |
| ----------- | --------------------------------------- |
| dev         | fewer replicas, debug configs           |
| staging     | moderate replicas, testing configs      |
| prod        | high replicas, production-ready configs |

---

## Overlay Features

Overlays can include:

* Patches (modify specific fields)
* Image transformations (change tags)
* Labels/annotations
* Namespace changes
* Resource additions or removals

---

## Benefits of Overlays

* No duplication of base YAML files
* Clear separation of environments
* Easy to maintain and scale
* Works well with CI/CD pipelines

---

## Summary

* Overlays customize base Kubernetes configurations per environment
* They extend (not replace) the base
* Enable clean, reusable, and scalable configuration management


# Scenarios

## Scenario 1
Update the api image in the `api-deployment` to use `caddy` docker image in the `QA` environment.

Perform this using an inline `JSON6902` patch.

Note: Please ensure to apply the updated config for `QA` environment before validation.

## Solution
In the `kustomization.yaml` file which is located at the `/root/code/k8s/overlays/QA` directory, add a `json6902` patch to `update` the image as shown below:

`overlays/QA/kustomization.yaml`

```yaml
patches:
  - target:
      kind: Deployment
      name: api-deployment
    patch: |-
      - op: replace
        path: /spec/template/spec/containers/0/image
        value: caddy
```

After modifications, apply the changes to create updated deployments in `QA` environment:

```bash
kubectl apply -k /root/code/k8s/overlays/QA
```

## Scenario 2
A mysql database needs to be added only in the `staging` environment.

Create a mysql deployment in a file called `mysql-depl.yaml` and define the deployment name as `mysql-deployment`.

Deploy `1` replica of the `mysql` container using `mysql` image and set the following env variables:

```yaml
- name: MYSQL_ROOT_PASSWORD
  value: mypassword
```

NOTE: Please ensure to deploy the changes committed in the `staging` environment before validation

## Solution
First let's create `overlays/staging/mysql-depl.yaml` as per the requirements:


```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      component: mysql
  template:
    metadata:
      labels:
        component: mysql
    spec:
      containers:
        - name: mysql
          image: mysql
          env:
            - name: MYSQL_ROOT_PASSWORD
              value: mypassword
```

Now update the staging environment kustomization.yaml file to include the deployment file we created earlier:

`overlays/staging/kustomization.yaml`


```yaml
resources:
  - mysql-depl.yaml
```

Applying the changes to k8s cluster for `staging` environment:


```bash
kubectl apply -k /root/code/k8s/overlays/staging
```
