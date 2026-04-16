## What are Patches in Kustomize?

**Patches** in Kustomize are a way to **modify specific parts of Kubernetes resources** in a precise and controlled manner.

Unlike transformers (which apply global changes), patches allow you to:
- Target specific resources
- Modify only certain fields
- Apply fine-grained (“surgical”) changes

---

## Why Use Patches?

Use patches when:
- You want to modify only one resource (not all)
- You need to update specific fields (e.g., replicas, labels, containers)
- You want environment-specific overrides without duplicating full YAML files

---

## Patch Components

Every patch requires:

1. **Operation**
   - `add`, `remove`, `replace`

2. **Target**
   - Defines which resource to modify:
     - `kind`
     - `name`
     - `namespace`
     - `labels`, etc.

3. **Value**
   - The new value (only for `add` and `replace`)

---

## Example 1: Replace Deployment Name (JSON 6902)

### `kustomization.yaml`

```yaml
patches:
- target:
    kind: Deployment
    name: api-deployment
  patch: |-
    - op: replace
      path: /metadata/name
      value: web-deployment
```

### Explanation
* Targets: Deployment named `api-deployment`
* Operation: `replace`
* Path: `/metadata/name`
* Result: Renames the deployment to `web-deployment`


## Example 2: Update Replicas
```yaml
patches:
    - target:
        kind: Deployment
        name: api-deployment
      patch: |-
        - op: replace
          path: /spec/replicas
          value: 5 
```

## Types of Patches
1. JSON 6902 Patch
* Uses JSON patch syntax
* Very precise
* Uses `op`, `path`, `value`

2. Strategic Merge Patch
* Uses normal YAML structure
* Easier to read
* Merges with existing resource

## Example (`Strategic Merge patch`)
```yaml
patches:
    - patch: |-
        apiVersion: apps/v1
        kind: Deployment
        metadata:
            name: api-deployment
        spec:
            replicas: 5
```

--- 

## Different Types of Patches

### JSON 6902 patch - Inline
```yaml
patches:
    - target:
        kind: Deployment
        name: api-deployment
      patch: |-
        - op: replace
          path: /spec/replicas
          value: 5 
```

### JSON 6902 patch - Separate File
You have the `kustomization.yaml`:

```yaml
patches:
- path: replica-patch.yaml
  target:
    kind: Deployment
    name: nginx-deployment
```
and the `replica-patch.yaml` would be:
```yaml
- op: replace
  path: /spec/replicas
  value: 5
```


### Strategic Merge Patch - Inline
```yaml
patches:
    - patch: |-
        apiVersion: apps/v1
        kind: Deployment
        metadata:
            name: api-deployment
        spec:
            replicas: 5
```

### Strategic Merge Patch - Separate File
You have the `kustomization.yaml`:
```yaml
patches:
    - replica-patch.yaml
```

You have the `replica-patch.yaml`:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
    name: api-development
spec:
    replicas: 5
```

---

## Working with Dictionaries (Maps)

### Add Field - JSON 6902
For example:
`api-depl.yaml`
```yaml
apiVersion: apps/v1 
kind: Deployment 
metadata:
    name: api-deployment
spec:
    replicas: 1
    selector:
        matchLabels:
            component: api
    template: 
        metadata:
            labels:
                component: api
        spec:
            containers:
            - name: nginx
              image: nginx
```

`kustomization.yaml`
```yaml
patches:
    - target:
        kind: Deployment
        name: api-deployment
    
      patch: |-
        - op: add
          path: /spec/template/metadata/labels/org
          value: Andreas
```

### Add Field - Strategic Merge Patch 
For example:
`api-depl.yaml`
```yaml
apiVersion: apps/v1 
kind: Deployment 
metadata:
    name: api-deployment
spec:
    replicas: 1
    selector:
        matchLabels:
            component: api
    template: 
        metadata:
            labels:
                component: api
        spec:
            containers:
            - name: nginx
              image: nginx
```

`kustomization.yaml`
```yaml
patches:
    - label-patch.yaml
```

`label-patch.yaml`
```yaml
apiVersion: apps/v1 
kind: Deployment 
metadata:
    name: api-deployment
spec:
    template: 
        metadata:
            labels:
                org: Andreas
```


### Remove Field - JSON 6902
For example:
`api-depl.yaml`
```yaml
apiVersion: apps/v1 
kind: Deployment 
metadata:
    name: api-deployment
spec:
    replicas: 1
    selector:
        matchLabels:
            component: api
    template: 
        metadata:
            labels:
                component: api
                org: Andreas
        spec:
            containers:
            - name: nginx
              image: nginx
```

`kustomization.yaml`
```yaml
patches:
    - target:
        kind: Deployment
        name: api-deployment
    
      patch: |-
        - op: remove
          path: /spec/template/metadata/labels/org
```


### Remove Field - Strategic Merge Patch 
For example:
`api-depl.yaml`
```yaml
apiVersion: apps/v1 
kind: Deployment 
metadata:
    name: api-deployment
spec:
    replicas: 1
    selector:
        matchLabels:
            component: api
    template: 
        metadata:
            labels:
                component: api
                org: Andreas
        spec:
            containers:
            - name: nginx
              image: nginx
```

`kustomization.yaml`
```yaml
patches:
    - label-patch.yaml
```

`label-patch.yaml`
```yaml
apiVersion: apps/v1 
kind: Deployment 
metadata:
    name: api-deployment
spec:
    template: 
        metadata:
            labels:
                org: null
```

---

## Working with Lists

### Replace List - JSON 6902

For example:
`api-depl.yaml`
```yaml
apiVersion: apps/v1 
kind: Deployment 
metadata:
    name: api-deployment
spec:
    replicas: 1
    selector:
        matchLabels:
            component: api
    template: 
        metadata:
            labels:
                component: api
        spec:
            containers:
            - name: nginx
              image: nginx
```

`kustomization.yaml`
```yaml
patches:
    - target:
        kind: Deployment
        name: api-deployment
    
      patch: |-
        - op: replace
          path: /spec/template/spec/containers/0
          value:
            name: haproxy
            image: haproxy
```

Result:
* Adds a new container to the list

### Replace List - Strategic Merge Patch

For example:
`api-depl.yaml`
```yaml
apiVersion: apps/v1 
kind: Deployment 
metadata:
    name: api-deployment
spec:
    replicas: 1
    selector:
        matchLabels:
            component: api
    template: 
        metadata:
            labels:
                component: api
        spec:
            containers:
            - name: nginx
              image: nginx
```


`kustomization.yaml`
```yaml
patches:
    - label-patch.yaml
```


`label-patch.yaml`
```yaml
apiVersion: apps/v1 
kind: Deployment 
metadata:
    name: api-deployment
spec:
    template:
        spec:
            containers:
            - name: nginx
              image: haproxy
```


### Add List - JSON 6902
For example:
`api-depl.yaml`
```yaml
apiVersion: apps/v1 
kind: Deployment 
metadata:
    name: api-deployment
spec:
    replicas: 1
    selector:
        matchLabels:
            component: api
    template: 
        metadata:
            labels:
                component: api
        spec:
            containers:
            - name: nginx
              image: nginx
```

`kustomization.yaml`
```yaml
patches:
    - target:
        kind: Deployment
        name: api-deployment
      patch: |-
        - op: add
          path: /spec/template/spec/containers/-
          value:
            name: haproxy
            image: haproxy
```

### Add List - Strategic Merge Patch
For example:
`api-depl.yaml`
```yaml
apiVersion: apps/v1 
kind: Deployment 
metadata:
    name: api-deployment
spec:
    replicas: 1
    selector:
        matchLabels:
            component: api
    template: 
        metadata:
            labels:
                component: api
        spec:
            containers:
            - name: web
              image: nginx
```


`kustomization.yaml`
```yaml
patches:
    - label-patch.yaml
```


`label-patch.yaml`
```yaml
apiVersion: apps/v1 
kind: Deployment 
metadata:
    name: api-deployment
spec:
    template:
        spec:
            containers:
            - name: haproxy
              image: haproxy
```



### Remove List Item - JSON 6902
For example:
`api-depl.yaml`
```yaml
apiVersion: apps/v1 
kind: Deployment 
metadata:
    name: api-deployment
spec:
    replicas: 1
    selector:
        matchLabels:
            component: api
    template: 
        metadata:
            labels:
                component: api
        spec:
            containers:
            - name: nginx
              image: nginx
            - name: database
              image: mongo
```

`kustomization.yaml`
```yaml
patches:
    - target:
        kind: name: Deployment
        name: api-deployment
      patch: |-
        - op: remove
          path: /spec/template/spec/containers/1
```


### Remove List Item - Strategic Merge Patch
For example:
`api-depl.yaml`
```yaml
apiVersion: apps/v1 
kind: Deployment 
metadata:
    name: api-deployment
spec:
    replicas: 1
    selector:
        matchLabels:
            component: api
    template: 
        metadata:
            labels:
                component: api
        spec:
            containers:
            - name: web
              image: nginx
            - name: database
              image: mongo
```

`kustomization.yaml`
```yaml
patches:
    - label-patch.yaml
```


`label-patch.yaml`
```yaml
apiVersion: apps/v1 
kind: Deployment 
metadata:
    name: api-deployment
spec:
    template:
        spec:
            containers:
            - $patch: delete
              name: database
```

## Key Differences
| Feature     | JSON 6902 Patch       | Strategic Merge Patch |
| ----------- | --------------------- | --------------------- |
| Syntax      | JSON-style operations | YAML structure        |
| Precision   | Very high             | Moderate              |
| Readability | Lower                 | Higher                |
| Best For    | Exact changes         | Simple overrides      |


## Summary
* Patches allow fine-grained control over Kubernetes manifests
* Two types:
    * JSON 6902 (precise, operation-based)
    * Strategic Merge (YAML-based, easier to read)
* Useful for:
    * Updating replicas
    * Modifying containers
    * Adding/removing labels
    * Environment-specific changes

Patches are essential when transformers are too broad and you need targeted modifications.


# Scenarios

## Scenario 1
In `api-patch.yaml` create a *strategic merge* patch to remove the `memcached` container.

Here is the `api-depl.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      component: api
  template:
    metadata:
      labels:
        component: api
    spec:
      containers:
        - name: nginx
          image: nginx
        - name: memcached
          image: memcached
```

Here is the `kustomization.yaml`:

```yaml
resources:
  - mongo-depl.yaml
  - api-depl.yaml
  - mongo-service.yaml

patches:
  - path: api-patch.yaml
```


## Solution

`api-patch.yaml` file
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-deployment
spec:
  template:
    spec:
      containers:
        - $patch: delete
          name: memcached
```


Let's apply the config too:

```kubectl apply -k /root/code/k8s/```




## Scenario 2
Create an `inline json6902` patch in the `kustomization.yaml` file to `remove` the label `org: andreas` from the `mongo-deployment`.

The `mongo-depl.yaml` is:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongo-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      component: mongo
  template:
    metadata:
      labels:
        component: mongo
        org: Andreas
    spec:
      containers:
        - name: mongo
          image: mongo
```

## Solution

`kustomization.yaml`

```yaml
patches:
  - target:
      kind: Deployment
      name: mongo-deployment
    patch: |-
      - op: remove
        path: /spec/template/metadata/labels/org
```

Let's apply the config too:
```kubectl apply -k /root/code/k8s/```



