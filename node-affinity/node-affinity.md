# Node Affinity

## What is Node Affinity?

**Node Affinity** is an advanced way to control **which nodes a Pod can be scheduled on**, based on node labels.

It is a more flexible and powerful version of `nodeSelector`.

---

## Node Selector vs Node Affinity

*pod-definition.yaml*
```
apiVersion: v1
kind: Pod
metadata:
    name: myapp-pod
spec:
    containers:
    - name: data-processor
      image: data-processor
    nodeSelector:
      size: Large
```

* Only supports exact match
* Cannot express complex logic (OR, NOT, preferences)


## Node Affinity (Advanced)
*pod-definition.yaml*
```
apiVersion: v1
kind: Pod
metadata:
    name: myapp-pod
spec:
    containers:
    - name: data-processor
      image: data-processor
    affinity:
      nodeAffinity:
        requiredDuringSchedulingIgnoredDuringExecution:
          nodeSelectorTerms:
          - matchExpressions:
            - key: size
              operator: In
              values:
              - Large
              - Medium
```

This allows:
* Multiple values (Large OR Medium)
* Advanced operators (In, NotIn, Exists)


## Node Affinity Operators



1. `In`
```
operator: In
values: [Large, Medium]
```

Node must have label `size=Large` OR `size=Medium`

2. `NotIn`
```
operator: NotIn
values: [Small]
```

Node must NOT have `size=Small`

3. `Exists`
```
operator: Exists
```

Node must have the label `size` (any value)


**What if NO node has the label?**

This depends on the *Node Affinity Type*


## Node Affinity Types

1. `requiredDuringSchedulingIgnoredDuringExecution`

**Strict requirement**

* Scheduler MUST find a matching node
* If not -> Pod stays in `Pending` state

Use when placement is critical

2. `preferredDuringSchedulingIgnoredDuringExecution`

**Soft requirement (best effort)**

* Scheduler tries to find a matching node
* If not -> Pod is scheduled on any available node

Use when:
* Running the workload is more important than placement


## What do “DuringScheduling” and “DuringExecution” mean?

**DuringScheduling**

* The moment when Kubernetes decides where to place the Pod
* Affinity rules are evaluated here

**DuringExecution**
* After the Pod is already running on a node

Important behavior:
* **Node affinity is NOT enforced after scheduling**

## What happens if node labels change?

Example:

* Pod scheduled on node with `size=Large`
* Admin removes label `size=Large`

Result:

* Pod continues running
* It is NOT evicted

Because:
`IgnoredDuringExecution`

## Summary of Behavior

| Type      | During Scheduling | During Execution |
| --------- | ----------------- | ---------------- |
| required  | MUST match        | Ignored          |
| preferred | Try to match      | Ignored          |


## Key Takeaways 
* Node Affinity = advanced node selection
* Supports:
    * OR conditions (`In`)
    * NOT conditions (`NotIn`)
    * existence checks (`Exists`)
* `required` -> strict placement
* `preferred` -> best effort
* Changes after scheduling -> do NOT affect running Pods


## When to Use Node Affinity?

Use it when you need:
* Flexible scheduling rules
* Multi-condition logic
* Better control than nodeSelector

## Pro Tip
* NodeSelector = simple cases
* NodeAffinity = production-level control




# Example - Exercise
Create a new deployment named `red` with the `nginx` image and `2` replicas, and ensure it gets placed on the `controlplane` node only.

Use the label key - `node-role.kubernetes.io/control-plane` - which is already set on the `controlplane` node.

## Solution:
### Create the file reddeploy.yaml file as follows:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: red
spec:
  replicas: 2
  selector:
    matchLabels:
      run: nginx
  template:
    metadata:
      labels:
        run: nginx
    spec:
      containers:
      - image: nginx
        imagePullPolicy: Always
        name: nginx
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: node-role.kubernetes.io/control-plane
                operator: Exists
```

Then run:
```kubectl create -f reddeploy.yaml```