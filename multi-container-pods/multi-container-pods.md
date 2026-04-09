# Multi-Container Pods

## What is a Multi-Container Pod?

A **multi-container Pod** is a Pod that runs **multiple containers together** as a single unit.

All containers in the same Pod:
- Share the **same network** (same IP & port space)
- Share **storage volumes**
- Are scheduled **together on the same node**
- Are tightly coupled and work as a **single application unit**



## Example:
`pod-defition.yaml`
```
apiVersion: v1
kind: Pod
metadata:
    name: simple-webapp
    labels:
        name: simple-webapp
spec:
    containers:
    - name: web-app
      image: web-app
      ports:
        - containerPort: 8080
    - name: main-app
      image: main-app
```

These two containers:
* Can communicate via localhost
* Run side-by-side inside the same Pod


## Multi-Container Design Patterns

1. Co-located Containers
2. Regular Init Containers
3. Sidecar Containers


## 1. Co-located Containers

Multiple containers running together simultaneously, each with a specific role.

### Use Case:
* App + helper container
* Web server + backend processor

### Example
```
apiVersion: v1
kind: Pod
metadata:
    name: simple-webapp
    labels:
        name: simple-webapp
spec:
    containers:
    - name: web-app
      image: web-app
      ports:
        - containerPort: 8080
    - name: main-app
      image: main-app
```

Both containers:
* Start at the same time
* Run continuously
* Work together as one system



## 2. Regular Init Containers

Containers that run before the main application starts

### Key Characteristics:
* Run sequentially
* Must complete successfully
* Only then -> main containers start

### Example
```
apiVersion: v1
kind: Pod
metadata:
    name: simple-webapp
    labels:
        name: simple-webapp
spec:
    initContainers:
    - name: db-checker
      image: busybox
      command: ["sh", "-c", "wait-for-db-to-start.sh"]
    - name: api-checker
      image: busybox
      command: ['sh', '-c', 'git clone <some-repository-that-will-be-used-by-application> ;']
    containers:
    - name: web-app
      image: web-app
      command: ['sh', '-c', 'echo The app is running! && sleep 3600']
      ports:
        - containerPort: 8080
```


Execution order:
1. db-checker
2. api-checker
3. web-app starts

### Use Cases:
* Wait for database readiness
* Run migrations
* Initialize configuration


## 3. Sidecar Containers

A helper container that runs alongside the main container to extend its functionality.

### Key Characteristics:
* Runs in parallel with main container
* Enhances or supports the main app
* Shares network and storage

### Common Use Cases:
* Logging (log shipper)
* Monitoring (metrics exporter)
* Proxy (service mesh sidecar like Envoy)

### Example
```
apiVersion: v1
kind: Pod
metadata:
    name: simple-webapp
    labels:
        name: simple-webapp
spec:
    containers:
    - name: web-app
      image: web-app
      ports:
        - containerPort: 8080
    initContainers:
    - name: log-shipper
      image: busybox
      command: 'setup-log-shipper.sh'
      restartPolicy: Always
```

## Co-located vs Sidecar Containers

| Feature   | Co-located              | Sidecar                |
| --------- | ----------------------- | ---------------------- |
| Purpose   | Multiple app components | Support main container |
| Role      | Equal importance        | Helper/auxiliary       |
| Lifecycle | Same                    | Same                   |
| Example   | Frontend + Backend      | App + Logger           |


## Key Difference
* Co-located containers -> part of the core application logic
* Sidecar containers -> provide supporting functionality

## Key Takeaways
* Multi-container Pods = tightly coupled containers
* Share network (localhost) and volumes
* 3 patterns:
    * Co-located -> multiple main components
    * Init -> run before app starts
    * Sidecar -> enhance app behavior

**Most common real-world pattern:**

**App + Sidecar (logging / monitoring / proxy)**

# Scenarios
## Scenario 1
The application outputs logs to the file `/log/app.log`. View the logs and try to identify the user having issues with Login.

Inspect the log file located inside the pod by utilizing the `kubectl exec` command.
## Solution
Run the command: `kubectl -n elastic-stack exec -it app -- cat /log/app.log`


## Scenario 2

The `app` pod in the `elastic-stack` namespace currently writes logs to `/log/app.log`.
Your task is to add a sidecar container that will ship these logs to Elasticsearch.

Requirements:

* Add a sidecar container named `sidecar` to the existing `app` pod.
* Use the image: `andreas/filebeat-configured`.
* Mount the log volume: The existing `log-volume` must be mounted to the sidecar container at `/var/log/event-simulator/`.
* Implementation: Define the sidecar as a Kubernetes native sidecar container using `initContainers`, and set the `restartPolicy` to `Always`.

Important Notes:

* You will need to delete and re-create the pod to add the sidecar container.
* Do not modify the existing app container or volume configuration.
* The sidecar should be defined as an `initContainer` and must run continuously alongside the main application container

Reference Documentation:

* Sidecar Containers: https://kubernetes.io/docs/concepts/workloads/pods/sidecar-containers/

Note: State persistence concepts are discussed in detail later in this course. For now, follow the pattern shown in the reference documentation.

## Solution

Utilize the manifest below to re-create the `app` pod with the updated configuration:
```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    name: app
  name: app
  namespace: elastic-stack
spec:
  initContainers:
  - name: sidecar
    image: andreas/filebeat-configured
    restartPolicy: Always
    volumeMounts:
      - name: log-volume
        mountPath: /var/log/event-simulator

  containers:
  - image: andreas/event-simulator
    name: app
    resources: {}
    volumeMounts:
    - mountPath: /log
      name: log-volume

  volumes:
  - hostPath:
      path: /var/log/webapp
      type: DirectoryOrCreate
    name: log-volume
```