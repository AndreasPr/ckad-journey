## Kubernetes Resources (CPU & Memory)

---

## CPU Resource

### What does 1 CPU mean?

In Kubernetes, **CPU is measured in cores (vCPU)**.

**1 CPU = 1 vCPU**, which corresponds to:
- 1 AWS vCPU  
- 1 Google Cloud Core  
- 1 Azure vCore  
- 1 Hyperthread on a physical CPU  

---

### 🔹 CPU Units

CPU can be expressed in **millicores (m)**:

- `1000m = 1 CPU`
- `500m = 0.5 CPU`
- `100m = 0.1 CPU`

Example:
```yaml
cpu: 250m
```

---

## Memory Resource
Memory is measured in bytes, typically using binary units:

Binary Units (Most common in Kubernetes)
* 1 Ki = 1,024 bytes
* 1 Mi = 1,048,576 bytes
* 1 Gi = 1,073,741,824 bytes

Decimal Units
* 1K = 1,000 bytes
* 1M = 1,000,000 bytes
* 1G = 1,000,000,000 bytes

Example: memory: 256Mi

≈ 268 MB

## Requests vs Limits

Each container can define:

Requests
* Minimum guaranteed resources
* Used by the scheduler to place Pods on nodes

Limits
* Maximum resources a container can use

### For example (pod-definition.yaml):
```
apiVersion: v1
kind: Pod
metadata:
    name: simple-webapp
    labels:
        name: simple-webapp
spec:
    containers:
    - name: simple-webapp
      image: simple-webapp
      ports:
        - containerPort: 8080
      resources:
        requests:
            memory: "1Gi"
            cpu: 2
        limits:
            memory: "2Gi"
            cpu: 2
```

## What happens when containers want to exceed limits?
CPU
* CPU is throttled (slowed down)
* Container is NOT killed
Memory
* Container is terminated (OOMKilled)
* Because memory cannot be throttled safely


## Default Behavior (if not set)
* If requests are not defined -> Kubernetes may assume defaults (via LimitRange or 0)
* If limits are not defined:
    * CPU -> unlimited (can use all available CPU)
    * Memory -> can grow until node runs out -> then gets killed



## Scenarios (2 Pods Example)

Assume a node with limited resources.

**1. No Requests – No Limits**

Behavior:
- Scheduler doesn’t reserve resources
- Pods can consume as much as available

Risk:
- Resource starvation
- One Pod can take everything


**2. No Requests – Limits**

Behavior:

- Scheduler doesn’t reserve resources
- But Pods cannot exceed limits

Risk:

- Pod might not get enough CPU/memory during contention


**3. Requests – Limits**

Behavior:

- Scheduler reserves resources
- Pod is guaranteed minimum
- Cannot exceed limits

This gives:

- Stability
- Predictability
- Fair resource usage

**4. Requests – No Limits**

Behavior:

- Guaranteed minimum resources
- BUT can consume unlimited CPU

Risk:

- Pod may starve others (especially CPU)



### LimitRange

A LimitRange sets default and allowed values for Pods in a namespace.

Important:

- Applies only at Pod creation time
- Does NOT affect existing Pods


#### Example (CPU) 
*limit-range-cpu.yaml*
```
apiVersion: v1
kind: LimitRange
metadata:
    name: cpu-resource-constraint
spec:
    limits:
    - default:
        cpu: 500m
      defaultRequest:
        cpu: 500m
      max:
        cpu: "1"
      min:
        cpu: 100m
      type: Container
```

What it enforces:

* Default request/limit = 500m
* Max CPU = 1
* Min CPU = 100m


#### Example (Memory)
*limit-range-memory.yaml*
```
apiVersion: v1
kind: LimitRange
metadata:
    name: memory-resource-constraint
spec:
    limits:
    - default:
        memory: 1Gi
      defaultRequest:
        memory: 1Gi
      max:
        memory: 1Gi
      min:
        memory: 500Mi
      type: Container
```

### ResourceQuota
* A ResourceQuota enforces limits at the namespace level.
* It controls total consumption across ALL Pods.

```
apiVersion: v1
kind: ResourceQuota
metadata:
    name: my-resource-quota
spec:
    hard:
        requests.cpu: 4
        requests.memory: 4Gi
        limits.cpu: 10
        limits.memory: 10Gi
```

Meaning:
* Total CPU requests ≤ 4
* Total memory requests ≤ 4Gi
* Total CPU limits ≤ 10
* Total memory limits ≤ 10Gi