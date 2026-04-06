## What are Image Transformers in Kustomize?

**Image transformers** in Kustomize are used to **modify container image names, tags, or registries** across Kubernetes manifests **without editing the original YAML files**.

They are especially useful for:
- Updating image versions (e.g., during deployments)
- Switching between registries (e.g., Docker Hub → private registry)
- Promoting the same app across environments (dev → staging → prod)

---

## Why Use Image Transformers?

Instead of manually updating image references in multiple files like:

```yaml
image: nginx:1.19
````

You can define the change **once** in `kustomization.yaml`, and Kustomize will apply it everywhere.

---

## Basic Example

### Base Deployment (`deployment.yaml`)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  template:
    spec:
      containers:
      - name: nginx
        image: nginx:1.19
```

---

### Kustomization File (`kustomization.yaml`)

```yaml
resources:
- deployment.yaml

images:
- name: nginx
  newTag: "1.21"
```

---

### Result After `kustomize build`

```yaml
containers:
- name: nginx
  image: nginx:1.21
```

---

## Full Image Override (Name + Tag)

You can replace both the image name and tag.

### Example

```yaml
images:
- name: nginx
  newName: myregistry.com/custom-nginx
  newTag: "2.0"
```

### Result

```yaml
image: myregistry.com/custom-nginx:2.0
```

---

## Multiple Images Example

```yaml
images:
- name: frontend
  newTag: "v2"
- name: backend
  newName: myrepo/backend-service
  newTag: "v5"
```

---

## Real-World Use Case

### Base (shared across environments)

```yaml
image: myapp:latest
```

### Dev Overlay

```yaml
images:
- name: myapp
  newTag: dev
```

### Prod Overlay

```yaml
images:
- name: myapp
  newTag: prod
```

This allows you to:

* Keep base configs unchanged
* Customize deployments per environment cleanly

---

## Key Fields

| Field   | Description                          |
| ------- | ------------------------------------ |
| name    | Original image name to match         |
| newName | (Optional) New image repository/name |
| newTag  | (Optional) New image tag/version     |

---

## Summary

* Image transformers let you update container images dynamically
* Defined in `kustomization.yaml`
* Avoid modifying base YAML files
* Ideal for CI/CD pipelines and environment-specific deployments

