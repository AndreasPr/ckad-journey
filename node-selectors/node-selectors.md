# Node Selector

## What is a Node Selector?

A **nodeSelector** is the simplest way to control **which node a Pod gets scheduled on**.

It works by matching **labels on nodes** with **selectors defined in the Pod**.

---

## Example

**pod-definition.yaml**
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

**How It Works**
* nodeSelector specifies a key-value pair
* Kubernetes scheduler looks for nodes with matching labels
* Pod is scheduled only on nodes that match

In this example:
* Pod will run only on nodes labeled: `size=Large`



## Step-by-Step Usage
### Step 1: Label a Node
```kubectl label nodes {node-name} {key}={value}```

**Example:**

```kubectl label nodes node-1 size=Large```

This adds a label to the node:
* Key -> size
* Value -> Large

### Step 2: Create the Pod
```kubectl create -f pod-definition.yaml```

The scheduler will:
* Find nodes with size=Large
* Place the Pod on one of them


## Important Notes
* Labels must exist before creating the Pod
* If no matching node is found -> Pod stays in `Pending` state
* Matching is exact (case-sensitive)

### Limitations of Node Selector

Node selectors are very simple and only support exact matches.

You CANNOT express conditions like:

* "Place Pod on `Large` OR `Medium` nodes"
* "Place Pod on nodes that are NOT `Small`"
* "Prefer one node type over another"

### When to Use Something More Advanced?

For complex scheduling rules, use:

* Node Affinity -> advanced rules (AND, OR, NOT, preferences)
* Pod Affinity / Anti-Affinity -> control Pod-to-Pod placement

## Key Takeaways
* Node Selector = simple label-based scheduling
* Requires pre-labeled nodes
* Supports exact match only
* Limited flexibility -> use Node Affinity for advanced cases