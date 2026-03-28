# Liveness Probes

## What is a Liveness Probe?

A **liveness probe** checks whether a container is **still alive and healthy**.

If the liveness probe **fails**:
- Kubernetes **restarts the container automatically**

---

## Why Use Liveness Probes?

Applications can:
- Crash internally
- Get stuck (deadlock)
- Stop responding

Even if the container is running, it might be **unhealthy**

Liveness probes detect this and **recover automatically**



# Example (HTTP Probe)
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
      livenessProbe:
        httpGet:
          path: /api/ready
          port: 8080
```

Kubernetes:
* Sends HTTP requests to /api/ready
* If it fails repeatedly → container is restarted




## Types of Liveness Probes
### 1. HTTP Probe
```
livenessProbe:
  httpGet:
    path: /api/ready
    port: 8080
```

Use for:
* Web apps
* REST APIs

### 2. TCP Probe
```
livenessProbe:
  tcpSocket:
    port: 3306
```

Use for:
* Databases
* Services where port availability = health

### 3. Exec Command Probe
```
livenessProbe:
  exec:
    command:
      - cat
      - /app/is_ready
```

Use for:
* Custom health checks
* Internal application logic






## Probe Configuration Options
### `initialDelaySeconds`
ex: `initialDelaySeconds: 10`

* Wait before starting health checks
* Prevents killing the container during startup

### `periodSeconds`
ex: `periodSeconds: 5`

* How often to run the probe

### `failureThreshold`
ex: `failureThreshold: 8`

* Number of failures before restart

### Combined Example
```
livenessProbe:
  httpGet:
    path: /api/ready
    port: 8080
  initialDelaySeconds: 10
  periodSeconds: 5
  failureThreshold: 8
```

## What Happens When a Liveness Probe Fails?
* Probe fails multiple times
* Kubernetes marks container as unhealthy
* Container is terminated
* Container is restarted automatically




## Liveness vs Readiness (IMPORTANT)
| Feature        | Liveness Probe            | Readiness Probe           |
| -------------- | ------------------------- | ------------------------- |
| Purpose        | Is app alive?             | Is app ready for traffic? |
| Failure action | Restart container         | Remove from Service       |
| Use case       | Crash / deadlock recovery | Startup / traffic control |



## Key Takeaways 
* Liveness = health check + auto-recovery
* Failing probe -> container restart
* Prevents stuck or frozen apps
* Works with HTTP, TCP, Exec
* Must be configured carefully (avoid restart loops)


## Pro Tip

Always combine:

* Readiness Probe -> controls traffic
* Liveness Probe -> ensures recovery

Together -> high availability + resilience