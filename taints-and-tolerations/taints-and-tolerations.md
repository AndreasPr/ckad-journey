# Taints and Tolerations

Taints and tolerations work together to control **which Pods can be scheduled on which Nodes**.

Think of it as:
- **Taints** → applied on Nodes (repel Pods)
- **Tolerations** → applied on Pods (allow them to tolerate taints)

---

## Why Use Them?

- Dedicate nodes for specific workloads (e.g., GPU, DB)
- Prevent certain Pods from running on specific nodes
- Isolate critical workloads

---

## Add a Taint to a Node
```kubectl taint nodes {node-name} {key}={value}:{taint-effect}```

Example:
```kubectl taint nodes node1 app=blue:NoSchedule```

This means:
- Node `node1` will reject Pods
- Unless Pods tolerate `app=blue`


# Taint Effects
1. **NoSchedule**
- Pods will NOT be scheduled on the node
- Existing Pods are NOT affected
2. **PreferNoSchedule**
- Scheduler will try to avoid placing Pods
- But it’s NOT a strict rule
3. **NoExecute (Important)**
- This is the most powerful and strict effect.
- Behavior:
    - New Pods -> **NOT scheduled** on the node
    - Existing Pods -> **EVICTED (removed)** if they don’t tolerate the taint


## Deep Dive: NoExecute
### Scenario:
```kubectl taint nodes node1 app=blue:NoExecute```

What happens:

**Pods WITHOUT toleration:**
* Immediately evicted from the node
* Also cannot be scheduled in the future

**Pods WITH toleration:**
* They can stay only if they tolerate the taint

**Example:**

```
tolerations:
- key: "app"
  operator: "Equal"
  value: "blue"
  effect: "NoExecute"
```

### Toleration Seconds (VERY IMPORTANT)
You can control how long a Pod can stay before eviction:
```
tolerations:
- key: "app"
  operator: "Equal"
  value: "blue"
  effect: "NoExecute"
  tolerationSeconds: 60
```

Meaning:
* Pod will stay for 60 seconds
* Then gets evicted automatically

### Real Use Case

Used for:

* Node failures / maintenance
* Graceful eviction of workloads


## Example Pod with Toleration
```
apiVersion: v1
kind: Pod
metadata:
    name: myapp-pod
spec:
    containers:
    - name: nginx-container
      image: nginx
    tolerations:
    - key: "app"
      operator: "Equal"
      value: "blue"
      effect: "NoSchedule"
```


#### View Taints on a Node:
```kubectl describe node kubemaster | grep Taint```

Shows all taints applied to the node.



# Key Takeaways
* Taints = repel Pods from nodes
* Tolerations = allow Pods to be scheduled
* `NoSchedule` -> block new Pods
* `PreferNoSchedule` -> soft rule
* `NoExecute` -> evicts existing Pods + blocks new ones

**Best practice:**
* Use `NoExecute` for strict isolation or node failure handling



