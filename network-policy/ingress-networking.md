# Ingress in Kubernetes


## What is Ingress?

Ingress is a Kubernetes resource that manages **external access to services inside a cluster**, typically HTTP and HTTPS.

It provides:
- URL-based routing  
- Host-based routing  
- TLS (HTTPS) termination  
- Centralized entry point for traffic  

---

## Key Concepts

- **Ingress Controller** → The actual implementation (e.g., NGINX, HAProxy, Traefik)  
- **Ingress Resource** → The configuration (rules) that define routing  

Important:
Kubernetes does NOT include an Ingress Controller by default. You must deploy one.

---

## How Ingress Works

Flow: **Client → Ingress Controller → Ingress Rules → Service → Pod**


Steps:
1. External request reaches the Ingress Controller  
2. Controller evaluates Ingress rules  
3. Routes request to the appropriate Service  
4. Service forwards traffic to Pods  

---

## How Ingress Load Balances

Ingress does not directly load balance itself.

Instead:
- It routes traffic to a **Service**
- The Service performs **load balancing across Pods**

Example:
- Ingress → routes to `wear-service`
- Service → distributes traffic across multiple Pods (round-robin)

---

# Ingress Controller

An Ingress Controller is a **reverse proxy / load balancer running inside the cluster**.

Common options:
- NGINX  
- HAProxy  
- Traefik  
- Istio  
- GCE Load Balancer  

---

## Example: NGINX Ingress Controller

### Deployment

```
apiVersion: apps/v1
kind: Deployment
metadata:
    name: nginx-ingress-controller
spec:
    replicas: 1
    selector:
        matchLabels:
            name: nginx-ingress
    template:
        metadata:
            labels:
                name: nginx-ingress
        spec:
            containers:
            - name: nginx-ingress-controller
              image: registry.k8s.io/ingress-nginx/controller
              args:
              - /nginx-ingress-controller
              - --configmap=$(POD_NAMESPACE)/nginx-configuration
              env:
              - name: POD_NAME
                valueFrom:
                  fieldRef:
                    fieldPath: metadata.name
              - name:
                valueFrom:
                  fieldRef:
                    fieldPath: metadata.namespace
              ports:
              - name: http
                containerPort: 80
              - name: https:
                containerPort: 443
```

## ConfigMap (Controller Configuration)

Used to externalize configuration like:
* timeouts
* SSL settings
* logging

```
apiVersion: v1
kind: ConfigMap
metadata:
    name: nginx-configuration
```

## Service (Expose Ingress Controller)
```
apiVersion: v1
kind: Service
metadata:
    name: nginx-ingress
spec:
    type: NodePort
    ports:
    - port: 80
      targetPort: 80
      protocol: TCP
    - port:
      targetPort: 443
      protocol: TCP
    selector:
        name: nginx-ingress
```

This exposes the controller to external traffic


## Service Account
```
apiVersion: v1
kind: ServiceAccount
metadata:
    name: nginx-ingress-serviceaccount
```

### Why needed?
The Ingress Controller needs permissions to:
* Watch Ingress resources
* Watch Services and Endpoints
* Update configurations dynamically

This is done via:
* ServiceAccount
* Roles / ClusterRoles
* RoleBindings / ClusterRoleBindings


# Ingress Resource
Defines routing rules for external traffic.

## Basic Example (Default Backend)
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
    name: ingress-wear
spec:
    defaultBackend:
        service:
            name: wear-service
            port: 80
```

All traffic goes to wear-service.

## Ingress Rules

Rules define how traffic is routed.

### 1. Path-Based Routing

Behavior:
* `/wear` → wear-service
* `/watch` → watch-service

Our requirement is to handle all traffic coming to my my-online-store.com and route them based on URL path. So, we need a single rule for this since we are only handling traffic to a single domain name, which is my-online-store.com in this case. So, paths is an array of multiple items, one path for each URL:

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
    name: ingress-wear-watch
spec:
    rules:
    - http:
        paths:
        - path: /wear
          backend:
            service: 
                name: wear-service
                port: 80
        - path: /watch
          backend:
            service:
                name: watch-service
                port: 80
```

### 2. Host-Based Routing

Behavior:
* `wear.my-online-store.com` → wear-service
* `watch.my-online-store.com` → watch-service

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
    name: ingress-wear-watch
spec:
    rules:
    - host: wear.my-online-store.com
      http:
        paths:
        - path: /wear
            backend:
            service: 
                name: wear-service
                port: 80
    - host: watch.my-online-store.com
      http:
        paths:
        - path: /watch
          backend:
            service:
                name: watch-service
                port: 80
```

### 3. Combined Host + Path Routing

You can combine both:

* Domain + path

Example:

* `example.com/wear`
* `example.com/watch`


# Commands
## Create Ingress
```kubectl create -f ingress.yaml```

## List Ingress
```kubectl get ingress```

## Describe Ingress
```kubectl describe ingress {ingress-name}```

## Imperative Command
```kubectl create ingress ingress-test --rule="wear.my-online-store.com/wear*=wear-service:80"```


## List Resources Across All Namespaces
```kubectl get all -A```

## List Deployments Across All Namespaces
```kubectl get deploy -A```

## Display all Ingress resources across namespaces
```kubectl get ingress --all-namespaces```

```kubectl get ingress -A```

## Inspect the ingress controller's manifest by executing:
```kubectl get deploy ingress-nginx-controller -n ingress-nginx -o yaml```

Outputs full YAML of the Ingress Controller Deployment

Useful for:
* Debugging configuration
* Checking container image, args, environment variables
* Verifying replicas and labels

## Edit an Ingress Resource
```kubectl edit ingress {ingress-name} -n app-space```

Opens the Ingress resource in a live editor

Allows you to:
* Modify routing rules
* Update paths or hosts
* Change backend services

Changes are applied immediately after saving


## Key Takeaways
* Ingress = Layer 7 (HTTP/HTTPS) routing
* Requires an Ingress Controller
* Supports:
    * Path-based routing
    * Host-based routing
    * TLS termination
* Works with Services for load balancing

## Pro Tip
* Use Ingress for HTTP/HTTPS only
* Use LoadBalancer Service for TCP/UDP traffic
* In production, combine with:
    * TLS (HTTPS)
    * DNS
    * External load balancer

