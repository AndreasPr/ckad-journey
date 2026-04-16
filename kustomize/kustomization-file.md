# What is `kustomization.yaml`?

The `kustomization.yaml` file is the **central configuration file in Kustomize**.
It tells Kustomize:

* **Which Kubernetes resources to manage**
* **What transformations to apply to them**

Think of it as the **entry point** for building your final Kubernetes manifests.

---

## Example

```yaml
# kubernetes resources to be managed by kustomize
resources:
- nginx-deployment.yaml
- nginx-service.yaml

# Customizations that need to be made
commonLabels:
  company: AndreasCompany
```

---

## What Does It Do?

### 1. Defines Resources

```yaml
resources:
- nginx-deployment.yaml
- nginx-service.yaml
```

* Lists all YAML files Kustomize should process
* These are your **base manifests**
* Can also reference directories (e.g., `../base`)

---

### 2. Applies Transformations

```yaml
commonLabels:
  company: AndreasCompany
```

* Adds labels to **all resources**
* Automatically injected into:

  * metadata.labels
  * selectors (where applicable)

---

## How Kustomize Uses It

When you run:

```bash
kustomize build k8s
```

Kustomize will:

1. Look for `kustomization.yaml` inside the `k8s/` directory
2. Load all listed resources
3. Apply transformations (labels, patches, etc.)
4. Output the **final rendered YAML**

---

## Important Behavior

### `kustomize build` DOES NOT deploy

It only **generates output**:

```bash
kustomize build k8s/
```

Output:

```yaml
# Final combined and transformed YAML printed to terminal
```

---

### To Apply to Cluster

You must pipe it to `kubectl`:

```bash
kustomize build k8s/ | kubectl apply -f -
```

OR (preferred, since it's built-in):

```bash
kubectl apply -k k8s/
```

### To Delete with Kustomize
```bash
kustomize build k8s/ | kubectl delete -f -
```

OR (preferred, since it's built-in):
```bash
kubectl delete -k k8s/
```


---
## apiVersion & kind (optional properties) (RECOMMENDED)
```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

# kubernetes resources to be managed by kustomize
resources:
- nginx-deployment.yaml
- nginx-service.yaml

# Customizations that need to be made
commonLabels:
  company: AndreasCompany
```

---

## What Can Be Defined in `kustomization.yaml`?

### 1. Resources

```yaml
resources:
- deployment.yaml
- service.yaml
```

---

### 2. Common Labels

```yaml
commonLabels:
  env: prod
```

---

### 3. Name Prefix/Suffix

```yaml
namePrefix: dev-
nameSuffix: -v1
```

---

### 4. Patches (Modify Existing YAML)

```yaml
patches:
- path: replica-patch.yaml
```

---

### 5. ConfigMap Generator

```yaml
configMapGenerator:
- name: app-config
  literals:
  - APP_MODE=prod
```

---

### 6. Secret Generator

```yaml
secretGenerator:
- name: app-secret
  literals:
  - password=abc123
```

---

### 7. Images (Override Image Versions)

```yaml
images:
- name: nginx
  newTag: 1.21
```

---

## Key Responsibilities

The `kustomization.yaml` file:

* Aggregates multiple YAML files
* Applies environment-specific changes
* Produces a final deployable manifest

---

## Workflow Summary

1. Create base YAML files
2. Define `kustomization.yaml`
3. Run:

   ```bash
   kustomize build .
   ```
4. Apply:

   ```bash
   kubectl apply -k .
   ```

---

## Why It’s Important

Without `kustomization.yaml`:

* Kustomize doesn’t know what to process
* No transformations are applied

It is **mandatory** for Kustomize to work.

---


## Managing Kubernetes Manifests Across Multiple Directories (Kustomize)

As Kubernetes projects grow, manifest files tend to increase in number and complexity. Organizing them into multiple directories (e.g., by service or component) improves maintainability, but introduces challenges when applying configurations.

Kustomize provides a structured and scalable way to manage and deploy these distributed manifests.

---

## Problem

Initially, all manifests might exist in a single directory:

```
k8s/
├── api-depl.yaml
├── api-service.yaml
├── db-depl.yaml
├── db-service.yaml
```

You could apply them using:

```bash
kubectl apply -f k8s/
```

However, as the project grows, it becomes necessary to organize files into subdirectories:

```
k8s/
├── api/
│   ├── api-depl.yaml
│   ├── api-service.yaml
├── db/
│   ├── db-depl.yaml
│   ├── db-service.yaml
```

At this point, managing deployments manually becomes inefficient.

---

## Solution: Root-Level `kustomization.yaml`

Create a `kustomization.yaml` file at the root of the `k8s/` directory to explicitly define all resources:

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
- api/api-depl.yaml
- api/api-service.yaml
- db/db-depl.yaml
- db/db-service.yaml
```

### Apply configuration:

```bash
kubectl apply -k k8s/
```

or:

```bash
kustomize build k8s/ | kubectl apply -f -
```

---

## Scalable Approach: Nested Kustomizations

As the number of resources grows, maintaining a flat list in the root file becomes difficult. A better approach is to define a `kustomization.yaml` inside each subdirectory.

### Example:

#### `k8s/db/kustomization.yaml`

```yaml
resources:
- db-depl.yaml
- db-service.yaml
```

#### `k8s/api/kustomization.yaml`

```yaml
resources:
- api-depl.yaml
- api-service.yaml
```

### Root `kustomization.yaml`

Now, instead of listing individual files, reference entire directories:

```yaml
resources:
- api/
- db/
- cache/
- kafka/
```

---

## Benefits of Nested Kustomization

* Improves modularity and separation of concerns
* Enables team ownership per service (e.g., API team manages `api/`)
* Simplifies root configuration
* Scales easily as new services are added
* Encourages reusable and composable configurations

---

## Key Commands

### Build manifests (preview only)

```bash
kustomize build k8s/
```

### Apply manifests

```bash
kubectl apply -k k8s/
```

### Delete manifests

```bash
kubectl delete -k k8s/
```


---

## Interview Tips

### Q1: What is `kustomization.yaml`?

It is the main configuration file that defines:

* Resources
* Transformations
  used by Kustomize to generate Kubernetes manifests.

---

### Q2: Does it deploy resources?

No. It only **builds** manifests. Deployment is done via `kubectl`.

---

### Q3: What is the difference between `kubectl apply -f` and `-k`?

* `-f` → apply raw YAML
* `-k` → apply Kustomize-managed configuration

---

### Q4: Where should it be located?

Inside the directory you pass to:

```bash
kubectl apply -k <directory>
```

---

## Summary

`kustomization.yaml` is:

* The **brain of Kustomize**
* Defines **what to include** and **how to modify it**
* Used to generate final Kubernetes manifests



# Scenarios
## Scenario 1
How many directories have been `pre-defined` in the `k8s` directory?

## Solution
We could either count the directories one by one from the `file explorer` to the left of VS Code window or run the below command in the terminal:
```bash
controlplane ~/code/k8s ➜  ls /root/code/k8s | wc -l
```

## Scenario 2
Let's create a single `kustomization.yaml` file in the root of the `k8s` directory and import all resources defined for `db`, `message-broker`, `nginx` into it.

Please ensure to `apply` the config after creating kustomization.yaml file.

## Solution

Create a `kustomization.yaml` file with the following content under the `/root/code/k8s` directory:

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

# kubernetes resources to be managed by kustomize
resources:
  - db/db-config.yaml
  - db/db-depl.yaml
  - db/db-service.yaml
  - message-broker/rabbitmq-config.yaml
  - message-broker/rabbitmq-depl.yaml
  - message-broker/rabbitmq-service.yaml
  - nginx/nginx-depl.yaml
  - nginx/nginx-service.yaml
```

Now let's apply the config:

```bash
controlplane ~/code/k8s ➜  pwd
/root/code/k8s

controlplane ~/code/k8s ➜  kubectl apply -k .
-----------OR---------------
controlplane ~/code ➜  kustomize build k8s/ | kubectl apply -f -
```


## Scenario 3
How many pods were deployed?

In the current(default) namespace.


## Solution
We can count the number of pods deployed using the below command in the terminal:

```kubectl get pods --no-headers | wc -l```


## Scenario 4
What is the `type` of the service that has been deployed for the `message-broker`?

## Solution

In the `rabbitmq-service.yaml` file, the service is defined as a `clusterIP` service.


```bash
controlplane code/k8s/message-broker ➜  cat rabbitmq-service.yaml 

apiVersion: v1
kind: Service
metadata:
  name: rabbit-cluster-ip-service
spec:
  type: ClusterIP
  selector:
    component: redis
  ports:
    - port: 5672
      targetPort: 5672
```

OR

We can list the services as follows, and look at the `Type` column for the service called `rabbit-cluster-ip-service`:
```bash
controlplane ~ ➜  kubectl get svc
```



## Scenario 5

Let's create a `kustomization.yaml` file in each of the `subdirectories` and import only the resources within that directory.

Please ensure to specify those directories within the root `kustomization.yaml` file.

NOTE: Don't forget to deploy the resources.

## Solution

We need to create `kustomization.yaml` in each of the subdirectories as follows:

```bash
controlplane ~/code ➜  tree
.
└── k8s
    ├── db
    │   └── kustomization.yaml
    ├── kustomization.yaml
    ├── message-broker
    │   └── kustomization.yaml
    └── nginx
        └── kustomization.yaml
4 directories, 12 files
```


`kustomization.yaml` file for `k8s/db`:
```yaml
resources:
  - db-depl.yaml
  - db-service.yaml
  - db-config.yaml
```


`kustomization.yaml` file for `k8s/message-broker`:
```yaml
resources:
  - rabbitmq-config.yaml
  - rabbitmq-depl.yaml
  - rabbitmq-service.yaml
```


`kustomization.yaml` file for `k8s/nginx`:
```yaml
resources:
  - nginx-depl.yaml
  - nginx-service.yaml
```


Root `kustomization.yaml` file for `k8s` directory:
```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

# kubernetes resources to be managed by kustomize
resources:
  - db/
  - message-broker/
  - nginx/
#Customizations that need to be made
```

Finally after defining each `kustomization.yaml` file let's apply our configuration:

```bash
controlplane ~/code ➜  kubectl apply -k /root/code/k8s/
--------------OR---------------
controlplane ~/code ➜  kustomize build /root/code/k8s/ | kubectl apply -f -
```