# Blue-Green Deployment in Kubernetes


## What is Blue-Green Deployment?

**Blue-Green Deployment** is a release strategy where you run **two identical environments**:

- **Blue** -> current live version  
- **Green** -> new version (staging/next release)

At any time:
- Only **one version serves production traffic**
- The other is used for **testing or standby**


## Goal

- Achieve **zero downtime deployments**
- Enable **instant rollback**
- Safely test new versions before release


# How It Works (Step-by-Step)


## Step 1: Deploy Current Version (Blue)

```
version: v1
image: myapp-image:1.0
```

Service points to:

```
selector:
  version: v1
```

**All traffic -> Blue (v1)**


## Step 2: Deploy New Version (Green)
```
version: v2
image: myapp-image:2.0
```

Important:
* Runs in parallel with Blue
* No production traffic yet

## Step 3: Test Green Environment

You can:
* Run internal tests
* Perform QA validation
* Use a separate test Service or port

Ensure everything works before switching

## Step 4: Switch Traffic

Update Service selector:
```
selector:
  version: v2
```

Result:
* Traffic instantly shifts from Blue → Green
* No downtime

## Step 5: Keep or Remove Blue

Options:
* Keep Blue -> for quick rollback
* Delete Blue -> free resources


## Example

We have the following `myapp-blue.yaml` deployment definition:

```
apiVersion: apps/v1 
kind: Deployment 
metadata:
    name: myapp-blue
    labels:
        app: myapp
        type: front-end
spec:
    template: 
        metadata:
            name: myapp-pod
            labels:
                version: v1
        spec:
            containers:
            - name: app-container
              image: myapp-image:1.0
    replicas: 5
    selector:
        matchLabels:
            version: v1
```

And we have the following `service-definition.yaml`:

```
apiVersion: v1
kind: Service
metadata:
    name: my-service
spec:
    selector:
        version: v1
```

Later, we deploy a newer version and we call it as green new separate deployment in `myapp-green.yaml`:
```
apiVersion: apps/v1 
kind: Deployment 
metadata:
    name: myapp-green
    labels:
        app: myapp
        type: front-end
spec:
    template: 
        metadata:
            name: myapp-pod
            labels:
                version: v2
        spec:
            containers:
            - name: app-container
              image: myapp-image:2.0
    replicas: 5
    selector:
        matchLabels:
            version: v2
```

## Advantages

* Zero downtime
* Instant rollback (just switch back selector)
* Safe testing in production-like environment
* Simple and predictable

## Disadvantages

- Requires double resources (Blue + Green)
- Database/schema changes can be tricky
- Not gradual (all-or-nothing switch)

## Rollback Strategy

If something goes wrong:
```
selector:
  version: v1
```

Traffic instantly returns to **Blue**

## Blue-Green vs Rolling Update
| Feature        | Blue-Green            | Rolling Update |
| -------------- | --------------------- | -------------- |
| Traffic Shift  | Instant               | Gradual        |
| Downtime       | None                  | None           |
| Resource Usage | High (2 environments) | Lower          |
| Rollback       | Instant               | Slower         |


## Key Takeaways 
* Two environments: Blue (live) & Green (new)
* Switch traffic via Service selector
* Enables instant rollback
* Requires extra resources
* Best for critical systems needing zero risk

## Pro Tip (Real-World)

In production, Blue-Green is often implemented with:
* Load balancers (instead of just Services)
* Ingress controllers
* Service meshes (Istio, Linkerd)
 
## When to Use Blue-Green?

Use it when:

* Downtime is unacceptable
* You need fast rollback
* You can afford duplicate environments



















# Canary Deployment in Kubernetes


## What is Canary Deployment?

A **Canary Deployment** is a release strategy where a **new version of an application is rolled out to a small subset of users first**, before exposing it to everyone.

Instead of switching all traffic at once (like Blue-Green):
- You **gradually shift traffic** to the new version


## Goal

- Reduce risk of failures  
- Detect issues early  
- Gradually roll out new features  


# How It Works


## Step 1: Deploy Primary Version (Stable)

```
version: v1
replicas: 5
```

This handles most of the traffic

## Step 2: Deploy Canary Version
```
version: v2
replicas: 1
```

This handles a small portion of traffic

## Step 3: Service Routes Traffic
```
selector:
  app: front-end
```

Important:

* Service selects both versions
* Because both have:
```
app: front-end
```

## Step 4: Traffic Distribution

Kubernetes distributes traffic:

* Based on number of Pods (replicas)
* Not exact %, but approximate



## How Traffic Percentage Works

Example:
* Primary (v1) -> 5 Pods
* Canary (v2) -> 1 Pod

Traffic split:
* v1 -> ~83%
* v2 -> ~17%

### How to Adjust Traffic Percentage?

You control traffic by adjusting replicas

### Increase Canary Traffic
```kubectl scale deployment myapp-canary --replicas=2```

Now:

* v1 -> 5 Pods
* v2 -> 2 Pods

Traffic:

* v1 -> ~71%
* v2 -> ~29%

### Full Rollout
```kubectl scale deployment myapp-canary --replicas=5```
```kubectl scale deployment myapp-primary --replicas=0```

Now:

* All traffic -> v2

### Important Limitation

Kubernetes Service:
* Uses **round-robin** load balancing
* Cannot control exact percentages (like 10%, 20%)





## Advanced Traffic Control (Production)

To get precise traffic splitting, use:
* Ingress controllers (NGINX, Traefik)
* Service Mesh (Istio, Linkerd)

These allow:
* Exact % routing (e.g., 90% / 10%)
* Header-based routing
* A/B testing

## Canary vs Blue-Green
| Feature        | Canary   | Blue-Green |
| -------------- | -------- | ---------- |
| Traffic shift  | Gradual  | Instant    |
| Risk           | Low      | Medium     |
| Rollback       | Easy     | Instant    |
| Resource usage | Moderate | High       |



## Key Takeaways
* Canary = gradual rollout
* Traffic split based on replica count
* Same Service -> selects both versions
* Monitor before full rollout
* For precise control -> use Ingress / Service Mesh

## Real-World Flow
1. Deploy v2 with small replicas
2. Monitor logs, metrics, errors
3. Gradually increase traffic
4. If stable -> promote v2
5. If issues -> scale down canary

## Pro Tip

Combine Canary with:
* Metrics Server + HPA
* Logging & monitoring

Enables safe, automated deployments



## Example
We have the following `myapp-primary.yaml` deployment definition:

```
apiVersion: apps/v1 
kind: Deployment 
metadata:
    name: myapp-primary
    labels:
        app: myapp
        type: front-end
spec:
    template: 
        metadata:
            name: myapp-pod
            labels:
                version: v1
                app: front-end
        spec:
            containers:
            - name: app-container
              image: myapp-image:1.0
    replicas: 5
    selector:
        matchLabels:
            app: front-end
```


And we have the following `service-definition.yaml`:

```
apiVersion: v1
kind: Service
metadata:
    name: my-service
spec:
    selector:
        app: front-end
```

Later, we deploy a newer version and we call it as new separate deployment in `myapp-canary.yaml`:
```
apiVersion: apps/v1 
kind: Deployment 
metadata:
    name: myapp-canary
    labels:
        app: myapp
        type: front-end
spec:
    template: 
        metadata:
            name: myapp-pod
            labels:
                version: v2
                app: front-end
        spec:
            containers:
            - name: app-container
              image: myapp-image:2.0
    replicas: 1
    selector:
        matchLabels:
            app: front-end
```
