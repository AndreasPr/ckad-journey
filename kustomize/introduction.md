# Kustomize in Kubernetes

## What is Kustomize?

**Kustomize** is a Kubernetes-native configuration management tool that lets you **customize raw YAML manifests without modifying the original files**.

It is built directly into `kubectl`, so you can use it without installing anything extra.

---

## What Problem Does Kustomize Solve?

In real-world environments, you often have:

* Same application deployed across **dev, staging, prod**
* Small differences between environments:

  * Replica count
  * Image tags
  * Config values

Without Kustomize:

* You duplicate YAML files
* Or manually edit them → error-prone

Kustomize solves this by:

* Keeping a **single base configuration**
* Applying **environment-specific overlays**

---

## Core Concept

Kustomize works with two main components:

### 1. Base

* Contains **default/shared configuration**
* Environment-agnostic
* Reusable across all environments

### 2. Overlays

* Environment-specific changes
* Modify or extend the base

---

## Example

### Base Configuration (`base/deployment.yaml`)

```yaml
apiVersion: apps/v1 
kind: Deployment 
metadata:
  name: nginx-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      component: nginx
  template: 
    metadata:
      labels:
        component: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
```

---

### Overlay: Dev (`overlays/dev`)

```yaml
spec:
  replicas: 1
```

---

### Overlay: Staging (`overlays/stg`)

```yaml
spec:
  replicas: 2
```

---

### Overlay: Production (`overlays/prod`)

```yaml
spec:
  replicas: 5
```

---

## Folder Structure

```
.
├── base/
│   ├── deployment.yaml
│   └── kustomization.yaml
├── overlays/
│   ├── dev/
│   │   └── kustomization.yaml
│   ├── stg/
│   │   └── kustomization.yaml
│   └── prod/
│       └── kustomization.yaml
```

---

## kustomization.yaml (Important File)

This file tells Kustomize:

* What resources to use
* What to modify

### Example: Base

```yaml
resources:
- deployment.yaml
```

---

### Example: Overlay (prod)

```yaml
resources:
- ../../base

patches:
- path: replica-patch.yaml
```

---

## How Kustomize Works

1. Load base resources
2. Apply overlays (patches, changes)
3. Generate final YAML
4. Apply to cluster

---

## Commands

### Apply with kubectl (built-in Kustomize)

```bash
kubectl apply -k overlays/prod
```

---

### Preview output

```bash
kubectl kustomize overlays/prod
```

---

## Key Features

### 1. Patching

Modify only specific fields instead of copying full YAML.

---

### 2. No Templating

* No variables like Helm (`{{ }}`)
* Pure YAML → easier to read and validate

---

### 3. Environment Separation

* Clean separation between base and environment-specific configs

---

### 4. Built-in Support

* Comes with `kubectl`
* No external dependencies required

---

## Kustomize vs Helm

| Feature     | Kustomize            | Helm          |
| ----------- | -------------------- | ------------- |
| Templates   | No                   | Yes           |
| Complexity  | Low                  | Medium/High   |
| YAML purity | Yes                  | No            |
| Packaging   | No                   | Yes           |
| Versioning  | No                   | Yes           |
| Use case    | Config customization | App packaging |

---

## When to Use Kustomize

Use Kustomize when:

* You want **simple customization**
* You prefer **plain YAML**
* You manage multiple environments
* You don’t need complex templating

---

## When NOT to Use Kustomize

Avoid Kustomize if:

* You need complex logic (loops, conditions)
* You need packaging/versioning → use Helm
* You want lifecycle automation → use Operators

---

## Advantages

* No duplication of YAML
* Clean and maintainable configs
* Native Kubernetes tool
* Easy to understand

---

## Limitations

* No advanced templating
* No built-in versioning
* Less powerful than Helm for large apps

---

## Install Kustomize

```bash
curl -s "https://raw.githubusercontent.com/kubernetes-sigs/kustomize/master/hack/install_kustomize.sh" | bash
```

Verify installation by running:
```bash
kustomize version --short
```

## Useful Questions

### Q1: What problem does Kustomize solve?

It eliminates duplication by allowing environment-specific customization of Kubernetes YAML without modifying the base configuration.

---

### Q2: What is the difference between base and overlay?

* Base → shared configuration
* Overlay → environment-specific modifications

---

### Q3: How do you apply a Kustomize configuration?

```bash
kubectl apply -k <directory>
```

---

### Q4: Does Kustomize use templating?

No. It works with **pure YAML patches**, not templates.

---

### Q5: Is Kustomize built into Kubernetes?

Yes. It is integrated into `kubectl`.

---

## Summary

Kustomize is:

* A **configuration customization tool**
* Based on **base + overlays**
* Uses **pure YAML (no templating)**
* Built into `kubectl`

It is ideal for:

* Managing multiple environments
* Keeping configurations clean and DRY (Don't Repeat Yourself)
