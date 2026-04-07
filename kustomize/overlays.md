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
