# ReplicaSet in Kubernetes

A **ReplicaSet** ensures that a specified number of identical Pods are running at all times in the cluster.

It continuously monitors the state of Pods and:

* Creates new Pods if the number falls below the desired count
* Deletes excess Pods if the number exceeds the desired count

## Example Definition

The following YAML defines a ReplicaSet that maintains **6 identical Pods**:

```yaml
apiVersion: apps/v1 
kind: ReplicaSet 
metadata:
  name: myapp-replicaset
  labels:
    app: myapp
    type: front-end
spec:
  replicas: 6
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

## Explanation

* **metadata.name**: Name of the ReplicaSet
* **spec.replicas**: Desired number of Pods (6)
* **spec.selector**: Defines how the ReplicaSet identifies which Pods it manages
* **spec.template**: Pod template used to create new Pods

  * Must match the selector labels
* **containers**: Defines the container(s) running inside each Pod

The ReplicaSet ensures that **exactly 6 Pods with label `type: front-end`** are always running.

---

# Common Commands

## Create

Create a ReplicaSet from a YAML file:

```bash
kubectl create -f replicaset-definition.yaml
```

---

## Get

List all ReplicaSets in the current namespace:

```bash
kubectl get replicaset
```

or

```bash
kubectl get rs
```

---

## Describe

View detailed information about a ReplicaSet:

```bash
kubectl describe replicaset myapp-replicaset
```

Includes:

* Pod status
* Events
* Labels
* Replica counts

---

## Explain

View documentation for the ReplicaSet resource:

```bash
kubectl explain replicaset
```

---

## Delete

Delete a ReplicaSet and all Pods it manages:

```bash
kubectl delete replicaset myapp-replicaset
```

---

## Edit (Live Configuration)

Edit the ReplicaSet directly in the cluster:

```bash
kubectl edit rs myapp-replicaset
```

Save and exit (vim):

```
:wq!
```

---

# Scaling Options

## 1. Update via File

Modify the YAML file and apply changes:

```bash
kubectl replace -f replicaset-definition.yaml
```

---

## 2. Scale Using File

Override replicas from the file:

```bash
kubectl scale --replicas=6 -f replicaset-definition.yaml
```

---

## 3. Scale Imperatively

Scale without modifying the YAML file:

```bash
kubectl scale --replicas=6 rs myapp-replicaset
```

---

# Key Notes

* The **selector and template labels must match**, otherwise the ReplicaSet will not manage the Pods correctly.
* ReplicaSets are typically **not used directly in production**.

  * Instead, they are managed by **Deployments**, which provide additional features like:

    * Rolling updates
    * Rollbacks
    * Versioning
