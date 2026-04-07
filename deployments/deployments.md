# Deployment in Kubernetes

A **Deployment** is a higher-level Kubernetes resource that manages **ReplicaSets** and ensures a specified number of Pods are running.

It provides additional capabilities such as:

* Rolling updates
* Rollbacks to previous versions
* Declarative updates
* Self-healing (via underlying ReplicaSets)

---

## Example Definition

The following YAML defines a Deployment that maintains **3 Pods** running an `nginx` container:

```yaml
apiVersion: apps/v1 
kind: Deployment 
metadata:
  name: myapp-deployment
  labels:
    app: myapp
    type: front-end
spec:
  replicas: 3
  selector:
    matchLabels:
      type: front-end
  template: 
    metadata:
      name: myapp-pod
      labels:
        app: myapp
        type: front-end
    spec:
      containers:
      - name: nginx-container
        image: nginx
```

---

## Explanation

* **metadata.name**: Name of the Deployment
* **spec.replicas**: Desired number of Pods (3)
* **spec.selector**: Identifies which Pods belong to this Deployment
* **spec.template**: Blueprint for creating Pods

  * Must match the selector labels
* **containers**: Defines the container(s) running inside each Pod

### How it works internally

* The Deployment creates a **ReplicaSet**
* The ReplicaSet creates and manages the Pods
* The Deployment manages the lifecycle of the ReplicaSet (updates, rollbacks)

---

# Common Commands

## Create

Create a Deployment from a YAML file:

```bash
kubectl create -f deployment-definition.yaml
```

---

## Get

List all Deployments:

```bash
kubectl get deployments
```

List all common Kubernetes resources:

```bash
kubectl get all
```

---

## Imperative Creation

Create a Deployment directly from the command line:

```bash
kubectl create deployment {name} --image={image-name} --replicas={number-of-replicas}
```

Example:

```bash
kubectl create deployment myapp --image=nginx --replicas=3
```

---

# Key Features of Deployments

## 1. Rolling Updates

When you update the Deployment (e.g., change image version), Kubernetes:

* Gradually replaces old Pods with new ones
* Ensures zero downtime (by default)

---

## 2. Rollbacks

You can revert to a previous version if something goes wrong:

```bash
kubectl rollout undo deployment myapp-deployment
```

---

## 3. Scaling

Scale the number of Pods:

```bash
kubectl scale deployment myapp-deployment --replicas=5
```

---

## 4. Status and History

Check rollout status:

```bash
kubectl rollout status deployment myapp-deployment
```

View rollout history:

```bash
kubectl rollout history deployment myapp-deployment
```

---

# Important Notes

* The **selector must match the Pod template labels**, otherwise the Deployment will not manage the Pods correctly.
* Deployments are the **recommended way** to manage stateless applications in Kubernetes.
* You should rarely create ReplicaSets directly—Deployments handle them for you.
