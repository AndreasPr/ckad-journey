# Metrics Server in Kubernetes

## What is Metrics Server?

The **Metrics Server** is a lightweight Kubernetes component that collects **resource usage metrics** (CPU & Memory) from nodes and Pods.

It provides:
- Real-time metrics for **kubectl top**
- Data for **Horizontal Pod Autoscaler (HPA)**

---

## Architecture Overview

- Each **Node** runs a **kubelet**
- The kubelet exposes resource usage data
- Metrics Server:
  - Collects metrics from all kubelets
  - Aggregates them
  - Exposes them via the Kubernetes API

Important:
- You typically have **ONE Metrics Server per cluster**

---

## What is cAdvisor?

**cAdvisor (Container Advisor)** is a component embedded inside the **kubelet**.

### Responsibilities:
- Collects **container-level metrics**
- Tracks:
  - CPU usage  
  - Memory usage  
  - Network usage  
  - Filesystem usage  

**Flow**: Container → cAdvisor → kubelet → Metrics Server → kubectl / HPA


---

## Installing Metrics Server

### Minikube
```minikube addons enable metrics-server```

### Other Clusters
```kubectl create -f deploy/1.8+/```

This deploys:

- Metrics Server Pod
- Required RBAC permissions
- API aggregation layer



## Viewing Metrics
### Node Metrics
`kubectl top node`

Shows:
- CPU usage per node
- Memory usage per node

### Pod Metrics
`kubectl top pod`

Shows:
- CPU usage per Pod
- Memory usage per Pod



## Important Limitations
* Metrics Server provides only current usage
* It does NOT store historical data
* It is NOT a full monitoring solution

## Production Monitoring
For advanced monitoring, use:

- Prometheus + Grafana
- Datadog
- Cloud-native monitoring tools

These provide:
- Historical data
- Alerts
- Dashboards

## Key Use Cases
- Horizontal Pod Autoscaler (HPA)
- Quick debugging (kubectl top)
- Resource usage visibility

## Key Takeaways
- Metrics Server = cluster-wide resource metrics provider
- Uses kubelet + cAdvisor for data collection
- Enables:
    - kubectl top
    - HPA
- Stores no historical data
- Lightweight and not a full monitoring solution



## Note
Deploy the Metrics Server in your Kubernetes cluster by applying the latest release components.yaml manifest using the following command:

```kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml```


# Scenario
## Scenario 1
Which node consumes the `most CPU(cores)`?

## Solution
`kubectl top node --sort-by='cpu' --no-headers | head -1`

Here we have used `head -1` command to print the node first in the sorted order, which is the one that uses the most cpu cores.

## Scenario 2
Identify the node that consumes the `most Memory(bytes)`.

## Solution
`kubectl top node --sort-by='memory' --no-headers | head -1`

Here we have used `head -1` command to print the node first in the sorted order, which is the one that uses the most Memory(bytes).

## Scenario 3
Identify the POD that consumes the `most Memory(bytes)` in default namespace.

## Solution
`kubectl top pod --sort-by='memory' --no-headers | head -1`

Here we have used `head -1` command to print the pod first in the order, which is the one that uses the most Memory(bytes).

## Scenario 4
Identify the POD that consumes the `least CPU(cores)` in default namespace.

## Solution
`kubectl top pod --sort-by='cpu' --no-headers | tail -1 `

Here we have used `tail -1` to list the last pod in the list, which is the pod that uses the least CPU(cores).
