# Service Types
1. NodePort: Exposes the Service on a static port on each node’s IP, allowing external access via NodeIP:NodePort.
2. ClusterIP: Default type. Exposes the Service only inside the cluster for internal communication between Pods.
3. LoadBalancer: Provisions an external load balancer (in cloud environments) to expose the Service publicly with a stable external IP.


## Service - NodePort (ex. service-definition.yaml)
This YAML defines a NodePort Service that exposes your application externally.
port (required) is the Service’s internal port inside the cluster, while targetPort is the port on the Pod (container).
nodePort: 30008 exposes the Service on each node’s IP at port 30008, allowing external access to Pods matching the selector (app: myapp, type: front-end).
```
apiVersion: v1
kind: Service
metadata:
    name: myapp-service
spec:
    type: NodePort
    ports:
        - targetPort: 80
          port: 80
          nodePort: 30008
    selector:
        app: myapp
        type: front-end
```

## Service - ClusterIP (ex. service-definition.yaml)
This YAML defines a ClusterIP Service named back-end:
port: 80 -> the Service port inside the cluster (required).
targetPort: 80 -> the port on the Pod that the Service routes traffic to.
selector -> chooses which Pods receive the traffic (app: myapp and type: back-end).
type: ClusterIP -> the Service is only accessible within the cluster, meaning external clients cannot reach it directly.
### Details about ClusterIP
- Default service type -> if you don’t specify type, Kubernetes creates a ClusterIP.
- Stable Endpoint -> While individual Pods are dynamic and ephemeral (meaning their IPs change when they are recreated or rescheduled), the ClusterIP remains constant for the lifetime of the Service, providing a reliable endpoint for other applications to connect to.
- Internal DNS -> Pods can access it using <service-name>.<namespace>.svc.cluster.local. Example: back-end.dev.svc.cluster.local.
- Load balancing -> automatically distributes traffic across all matching Pods.
- Use cases -> connecting microservices inside the cluster (e.g., front-end Pods talk to back-end Pods).
- Not exposed externally -> to expose outside, you need NodePort, LoadBalancer, or Ingress.

```
apiVersion: v1
kind: Service
metadata:
    name: back-end
spec:
    type: ClusterIP
    ports:
        - targetPort: 80
          port: 80
    selector:
        app: myapp
        type: back-end
```


## Commands
### Create the Service defined in your YAML file (e.g., NodePort or ClusterIP) in the cluster:
```kubectl create -f service-definition.yaml```
### List all Services in the current namespace, showing their type, cluster IP, external IP (if any), ports, and age:
```kubectl get services```
or 
```kubectl get svc```

### Show detailed information about the Service, including its selector, endpoints (Pods it routes to), ports, and events:
```kubectl describe service {name-of-service}```
or
```kubectl describe svc {name-of-service}```