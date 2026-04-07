# Namespace in Kubernetes

A **Namespace** is a logical partition within a Kubernetes cluster used to organize and isolate resources.

It allows multiple teams, environments, or applications to share the same cluster while remaining logically separated.

---

## Example Definition

The following YAML defines a Namespace named `dev`:

```yaml
apiVersion: v1 
kind: Namespace 
metadata:
  name: dev
```

---

## Why Use Namespaces?

* **Isolation**: Separate environments (e.g., dev, staging, prod)
* **Resource management**: Apply quotas and limits per namespace
* **Access control**: Use RBAC to restrict access per namespace
* **Organization**: Group related resources together

---

# ResourceQuota

A **ResourceQuota** limits the total amount of resources that can be consumed within a namespace.

---

## Example: Compute Quota

The following YAML defines a quota for the `dev` namespace:

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
  namespace: dev
spec:
  hard:
    pods: "10"
    requests.cpu: "4"
    requests.memory: 5Gi
    limits.cpu: "10"
    limits.memory: 10Gi
```

---

## Explanation

This quota enforces:

* Maximum **10 Pods**
* Maximum **4 CPU (requests)**
* Maximum **10 CPU (limits)**
* Maximum **5Gi memory (requests)**
* Maximum **10Gi memory (limits)**

If a new resource exceeds these limits, Kubernetes will reject the request.

---

## Apply the ResourceQuota

```bash
kubectl create -f compute-quota.yaml
```

---

# Internal DNS in Kubernetes

Kubernetes provides built-in DNS for service discovery.

## Example

```text
db-service.dev.svc.cluster.local
```

### Breakdown

* **db-service** → Service name
* **dev** → Namespace
* **svc** → Indicates a Service
* **cluster.local** → Default cluster DNS domain

### Purpose

This DNS name allows Pods to communicate with services across namespaces using a stable and predictable address instead of IPs.

---

# Common Commands

## Create Namespace

```bash
kubectl create -f namespace-dev.yaml
```

or

```bash
kubectl create namespace dev
```

---

## List Pods in a Namespace

```bash
kubectl get pods --namespace=dev
```

or

```bash
kubectl get pods -n dev
```

---

## Set Default Namespace

Set `dev` as the default namespace for future commands:

```bash
kubectl config set-context $(kubectl config current-context) --namespace=dev
```

---

## List Pods Across All Namespaces

```bash
kubectl get pods --all-namespaces
```

or

```bash
kubectl get pods -A
```

---

## List Namespaces

```bash
kubectl get namespaces
```

or

```bash
kubectl get ns
```

---

# Key Notes

* Namespaces provide **logical isolation**, not full security isolation by default.
* Combine Namespaces with:

  * **RBAC** for access control
  * **Network Policies** for traffic isolation
* Default namespaces include:

  * `default`
  * `kube-system`
  * `kube-public`
  * `kube-node-lease`
