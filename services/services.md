# Kubernetes Services

A **Service** in Kubernetes provides a stable network endpoint to access a set of Pods.

Since Pods are ephemeral (they can be created, destroyed, or rescheduled), Services ensure:

* Stable IP address
* DNS-based service discovery
* Built-in load balancing across Pods

---

# Service Types

## 1. NodePort

Exposes the Service on a static port on each node’s IP.

* Accessible externally via:
  `NodeIP:NodePort`
* Suitable for simple external access (mainly for testing or small setups)

---

## 2. ClusterIP (Default)

Exposes the Service **only داخل the cluster**.

* Used for internal communication between services
* Not accessible from outside the cluster

---

## 3. LoadBalancer

Creates an **external load balancer** (cloud provider required).

* Provides a public IP address
* Automatically routes traffic to the Service

---

# NodePort Service

## Example

```yaml id="nzt2jv"
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  type: NodePort
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30008
  selector:
    app: myapp
    type: front-end
```

---

## Explanation

* **type: NodePort** → Exposes the Service externally
* **port** → Service port داخل cluster (required)
* **targetPort** → Port on the Pod/container
* **nodePort** → Port exposed on each node (range: 30000–32767)
* **selector** → Matches Pods that will receive traffic

### Access

```
http://<NodeIP>:30008
```

---

# ClusterIP Service

## Example

```yaml id="bc1md1"
apiVersion: v1
kind: Service
metadata:
  name: back-end
spec:
  type: ClusterIP
  ports:
    - port: 80
      targetPort: 80
  selector:
    app: myapp
    type: back-end
```

---

## Explanation

* **type: ClusterIP** → Internal-only access
* **port** → Service port داخل cluster
* **targetPort** → Pod port
* **selector** → Defines target Pods

---

## Key Features of ClusterIP

### 1. Default Type

If no type is specified, Kubernetes creates a **ClusterIP Service**.

---

### 2. Stable Endpoint

* Pods are dynamic (IPs change)
* Service IP remains constant
* Provides a reliable access point

---

### 3. Internal DNS

Pods can access the service using:

```
<service-name>.<namespace>.svc.cluster.local
```

Example:

```
back-end.dev.svc.cluster.local
```

---

### 4. Load Balancing

* Automatically distributes traffic across all matching Pods
* Uses kube-proxy (iptables/IPVS)

---

### 5. Use Cases

* Microservices communication
* Backend APIs
* Internal databases

---

### 6. Not Externally Accessible

To expose outside the cluster, use:

* NodePort
* LoadBalancer
* Ingress

---

# Commands

## Create Service

```bash id="xg2eqb"
kubectl create -f service-definition.yaml
```

---

## List Services

```bash id="s4okcr"
kubectl get services
```

or

```bash id="p1r2gc"
kubectl get svc
```

---

## Describe Service

```bash id="93egj5"
kubectl describe service {service-name}
```

or

```bash id="3i7k8p"
kubectl describe svc {service-name}
```

---

# Key Notes

* Services use **label selectors** to dynamically discover Pods
* If no Pods match the selector → Service has **no endpoints**
* Services provide **load balancing at Layer 4 (TCP/UDP)**
* For advanced routing (HTTP paths, domains), use **Ingress**
