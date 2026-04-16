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


# Scenarios

## Scenario 1
What is the `label` that will get assigned to every Kubernetes resource within `/root/code/k8s/` project?

## Solution
The question asked for a `label` that will be assigned to every resource. This means a `commonLabels` transformer will be applied on the root `kustomization.yaml` file in `k8s` directory .i.e. `k8s/kustomization.yaml`

`k8s/kustomization.yaml`
```yaml
commonLabels:
  sandbox: dev
```


## Scenario 2
What is the name that will be `prefixed` before all database resources?

## Solution
All database resources includes both `sql` and `nosql` configs. Inside the `kustomization.yaml` file in the `db` directory the `namePrefix` transformer will be applied.

`k8s/db/kustomization.yaml`
```yaml
namePrefix: data-
```

## Scenario 3
What is the namespace that all the monitoring resources will be deployed to?

## Solution
The `kustomization.yaml` file in the `monitoring` directory has a `namespace` transformer set to `logging`.

`k8s/monitoring/kustomization.yaml`
```yaml
namespace: logging
```


## Scenario 4
Assign the following annotation to all `nginx` and `monitoring` resources:
```
owner: bob@gmail.com
```

## Solution
As we want to apply the annotation only to `nginx` and `monitoring` resources, we will modify the following files as shown below:

`k8s/nginx/kustomization.yaml` and `k8s/monitoring/kustomization.yaml`

```yaml
commonAnnotations:
  owner: bob@gmail.com
```

## Scenario 5
Transform all `postgres` images in the project to `mysql`.

## Solution
Since the requirement was to change all postgres images to mysql this means adding an `image` transformer to the `root kustomization.yaml` file.

`k8s/kustomization.yaml`

```yaml
images:
  - name: postgres
    newName: mysql
```


## Scenario 6
Transform all `nginx` images in the nginx directory to `nginx:1.23`.

## Solution
For this task, the `kustomization.yaml` file in the `nginx` directory needs to be modified with an `image` transformer.

Since the image itself isn’t changing and only a `new tag` is getting assigned, add the `newTag` property as shown below:

`k8s/nginx/kustomization.yaml`
```yaml
images:
  - name: nginx
    newTag: "1.23"
```