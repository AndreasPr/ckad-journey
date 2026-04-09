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



## Useful Scenarios

### 1. Scenario
Add an annotation to the web-app-ingress Ingress to redirect all HTTP requests to HTTPS.
Use the NGINX Ingress Controller annotation for SSL redirect.

### Solution

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-app-ingress
  namespace: webapp
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
spec:
  ingressClassName: nginx
  tls:
  - hosts:
      - app.andreas.local
    secretName: app-tls
  rules:
  - host: app.andreas.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-app
            port:
              number: 80
```

### 2. Scenario
Create a service to make ingress available to external users. Specs:
* Name: ignress
* Type: NodePort
* Port: 80
* TargetPort: 80
* NodePort: 30080
* Namespace: ingress-space
* Use the right selector

### Solution
```kubectl expose deploy ingress-controller -n ingress-space --name ingress --port=80 --target-port=80 --type NodePort```

After that, run the command: ```kubectl edit svc ingress -n ingress-space``` and update the field `nodePort` to the value `30080`.


### 3. Scenario
Create an ingress resource to make the apps available at `/watch` and `/wear` on the ingress service.
(We have already created two services `wear-service` and `video-service`)
### Solution
```kubectl create ingress ingress-wear-watch -n app-space --rule="/wear=wear-service:8080" -- rule="/watch=video-service:8080"```

Also, we should update the ingress by running the command: `kubectl edit ingress ingress-wear-watch -n app-space` and include the following under the `metadata` property:
```
annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
```




# Ingress Rewrite Target (NGINX)

---

## Overview

Different Ingress Controllers provide various options to customize request handling.  
One commonly used feature in the NGINX Ingress Controller is the **rewrite-target** annotation.

This option allows you to **modify the request URL path before forwarding it to the backend service**.

---

## Problem Scenario

Assume we have two applications:

- **Watch App** → serves content at:
```

http://<watch-service>:<port>/

```

- **Wear App** → serves content at:
```

http://<wear-service>:<port>/

```

---

## Requirement

We want users to access the applications via Ingress using:

```

http://<ingress-service>:<port>/watch → watch-service
http://<ingress-service>:<port>/wear  → wear-service

```

---

## What Happens Without Rewrite

Without using `rewrite-target`, the request path is forwarded **as-is**:

```

/watch → /watch
/wear  → /wear

```

So internally:

```

http://<ingress>/watch → http://<watch-service>/watch
http://<ingress>/wear  → http://<wear-service>/wear

````

---

## Problem

The backend applications:
- Do **NOT** expect `/watch` or `/wear`
- Only serve content at `/`

Result:
- Requests fail with **404 Not Found**

---

## Solution: Rewrite Target

We use the annotation:

```yaml
nginx.ingress.kubernetes.io/rewrite-target: /
````

This rewrites incoming paths:

```
/watch → /
/wear  → /
```

---

## How It Works

Think of it as a **search and replace**:

```
replace(<incoming-path>, <rewrite-target>)
```

Example:

```
replace("/watch", "/")
```

---

## Example Configuration

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: test-ingress
  namespace: critical-space
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - http:
      paths:
      - path: /pay
        pathType: Prefix
        backend:
          service:
            name: pay-service
            port:
              number: 8282
```

---

## Behavior

```
Incoming:  /pay
Rewritten: /
Forwarded: http://pay-service:8282/
```

---

# Advanced Rewrite (Regex)

Sometimes you want to preserve part of the URL.

---

## Example

```yaml
nginx.ingress.kubernetes.io/rewrite-target: /$2
```

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: rewrite
  namespace: default
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$2
spec:
  rules:
  - host: rewrite.bar.com
    http:
      paths:
      - path: /something(/|$)(.*)
        pathType: Prefix
        backend:
          service:
            name: http-svc
            port:
              number: 80
```

---

## Explanation

Path:

```
/something(/|$)(.*)
```

Matches:

* `/something`
* `/something/abc`
* `/something/anything/here`

---

## Rewrite Logic

```
/something(/|$)(.*) → /$2
```

Examples:

| Incoming Request  | Rewritten Path |
| ----------------- | -------------- |
| /something        | /              |
| /something/hello  | /hello         |
| /something/api/v1 | /api/v1        |

---

## Key Idea

* `$2` = everything after `/something`
* This allows flexible routing while preserving subpaths

---

# Key Takeaways

* `rewrite-target` modifies request paths before reaching backend
* Useful when backend apps do not expect prefixed paths
* Prevents 404 errors caused by mismatched routes
* Supports both:

  * Simple rewrites (`/ → /`)
  * Advanced regex-based rewrites

---

# Pro Tip

Always verify:

* Backend application routes
* Ingress path configuration
* Rewrite rules
