# Readiness Probes & Pod Conditions

## What are Readiness Probes?

A **readiness probe** checks whether a container is **ready to receive traffic**.

Important:
- If a Pod is **NOT ready** -> it is **removed from Service endpoints**
- Kubernetes will **NOT send traffic** to it

---

## Why Use Readiness Probes?

Applications often need time to:
- Start up
- Load data
- Connect to databases

Without readiness probes:
- Traffic may hit the app **too early** → failures


# Pod Conditions

Pod conditions describe the **current state of a Pod**.



### 1. `PodScheduled`
- Pod has been assigned to a node

---

### 2. `Initialized`
- All **init containers** have completed successfully

---

### 3. `ContainersReady`
- All containers inside the Pod are ready

---

### 4. `Ready` (MOST IMPORTANT)
- Pod is ready to **serve traffic**
- Controlled by **readiness probes**

---

# Types of Readiness Probes


## 1. HTTP Test

Sends an HTTP request to the container
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
      readinessProbe:
        httpGet:
          path: /api/ready
          port: 8080
```

Use Case:
* Web applications
* REST APIs

## 2. TCP Test
Checks if a port is open
```
readinessProbe:
    tcpSocket:
        port: 3306
```

Use Case:
* Databases (MySQL, PostgreSQL)
* Any TCP-based service

## 3. Exec Command
Runs a command inside the container
```
readinessProbe:
    exec:
        command:
            - cat
            - /app/is_ready
```
Use Case:
* Custom logic
* File existence checks

## Probe Configuration Options
### `initialDelaySeconds`

ex: `initialDelaySeconds: 10`

- Wait before starting checks
- Useful for slow-starting apps

### `periodSeconds`

ex: `periodSeconds: 5`

- How often to run the probe

### `failureThreshold`

ex: `failureThreshold: 8`

- Number of failures before marking Pod as Not Ready


### Example
```
readinessProbe:
    httpGet:
        path: /api/ready
        port: 8080
    initialDelaySeconds: 10
    periodSeconds: 5
    failureThreshold: 8
```

## What Happens When a Probe Fails?
* Pod is marked Not Ready
* Removed from Service load balancing
* Traffic stops going to it
* Container is NOT restarted (important!)


## Readiness vs Liveness (Quick Insight)

| Feature  | Readiness Probe      | Liveness Probe       |
| -------- | -------------------- | -------------------- |
| Purpose  | Can receive traffic  | Is container healthy |
| Failure  | Removed from service | Container restarted  |
| Use case | Startup/warm-up      | Deadlock/crash       |


## Key Takeaways
* Readiness = traffic control
* Determines if Pod is included in Service
* Does NOT restart containers
* Supports HTTP, TCP, Exec checks
* Essential for zero-downtime deployments

## Real-World Example

During deployment:

* New Pods start -> NOT ready yet
* Old Pods still serve traffic
* Once new Pods pass readiness -> traffic shifts

This enables rolling updates without downtime