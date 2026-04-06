## What are Transformers in Kustomize?

**Transformers** in Kustomize are built-in mechanisms that modify Kubernetes resource manifests **without changing the original YAML files**.

They allow you to apply **consistent, reusable changes across multiple resources**, such as adding labels, prefixes, namespaces, or annotations.

Instead of editing each manifest manually, you define transformations once in `kustomization.yaml`, and Kustomize applies them automatically during `build`.

---

## Common Transformers

### 1. `commonLabels`

Adds the same label(s) to **all resources** managed by Kustomize.

#### Use Case
- Environment tagging (dev, prod)
- App identification
- Monitoring / logging grouping

#### Example

```yaml
# kustomization.yaml
commonLabels:
  app: my-app
  env: production
````

#### Result (applied to all resources)

```yaml
metadata:
  labels:
    app: my-app
    env: production
```

---

### 2. `namePrefix` / `nameSuffix`

Adds a prefix or suffix to the **names of all resources**.

#### Use Case

* Distinguish environments (dev-, prod-)
* Avoid naming conflicts across clusters

#### Example

```yaml
# kustomization.yaml
namePrefix: dev-
nameSuffix: -v1
```

#### Original Resource

```yaml
metadata:
  name: nginx
```

#### Result

```yaml
metadata:
  name: dev-nginx-v1
```

---

### 3. `namespace`

Assigns a **common namespace** to all resources.

#### Use Case

* Deploy all resources into a specific environment namespace
* Avoid manually setting namespace in each YAML

#### Example

```yaml
# kustomization.yaml
namespace: staging
```

#### Result

```yaml
metadata:
  namespace: staging
```

Note:

* This overrides any namespace defined inside individual resource files.

---

### 4. `commonAnnotations`

Adds annotations to all resources.

#### Use Case

* Add metadata for tools (monitoring, tracing, CI/CD)
* Track deployment information

#### Example

```yaml
# kustomization.yaml
commonAnnotations:
  owner: team-platform
  managed-by: kustomize
```

#### Result

```yaml
metadata:
  annotations:
    owner: team-platform
    managed-by: kustomize
```

---

## Full Example

### `kustomization.yaml`

```yaml
resources:
- deployment.yaml
- service.yaml

commonLabels:
  app: my-app
  env: dev

namePrefix: dev-

namespace: dev-namespace

commonAnnotations:
  owner: dev-team
```

---

## Key Benefits of Transformers

* No duplication across YAML files
* Clean separation between base config and environment-specific changes
* Easy to scale across multiple environments
* Reduces human error in manual edits

---

## Summary

Transformers in Kustomize:

* Modify resources dynamically during build time
* Apply changes globally across all manifests
* Help maintain clean, DRY (Don't Repeat Yourself) configurations
