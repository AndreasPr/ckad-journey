# Docker Storage

---

## What is Docker Storage?

Docker storage refers to **how Docker manages and persists data** for:
- Images  
- Containers  
- Volumes  

Docker must handle:
- Application files (from images)  
- Runtime changes (from containers)  
- Persistent data (databases, logs, etc.)  

---

# Storage Drivers

## What are Storage Drivers?

Storage drivers control **how Docker stores and manages image layers and container filesystems**.

They are responsible for:
- Layering images  
- Managing file changes  
- Implementing Copy-on-Write (CoW)  

---

## Common Storage Drivers

- AUFS  
- ZFS  
- BTRFS  
- Device Mapper  
- Overlay  
- Overlay2 (default in most modern systems)  

---

## Why Storage Drivers Matter

They determine:
- Performance  
- Disk usage efficiency  
- How layers are combined  

---

# Volume Drivers

## What are Volume Drivers?

Volume drivers manage **how persistent data is stored and shared**.

They allow:
- Data persistence beyond container lifecycle  
- Integration with external storage systems  

Examples:
- Local (default)  
- NFS  
- Cloud storage (AWS EBS, Azure Disk)  

---

# How Docker Stores Data Locally

Docker stores data under:

```/var/lib/docker```

Inside this directory:

* Image layers
* Container layers
* Volumes

---

# Layered Architecture

## How Docker Stores Images

Docker images are built using **layers**.

Example:

```dockerfile
FROM ubuntu
RUN apt-get update
RUN apt-get install python
COPY app.py /app
```

Each instruction creates a **new layer**.

---

## Image Layers

* Read-only
* Cached and reused
* Shared across containers

---

## Container Layer

When a container runs:

* A new **read-write layer** is added on top

---

## Key Concept

```
Image Layers (read-only)
        +
Container Layer (read-write)
```

---

# Example: Build an Image

```docker build Dockerfile -t andreas/my-app```

Creates layered image:

* Base image
* Installed packages
* Application code

---

# What Happens When You Modify a Container?

If you run a container and create a file:

```touch temp.txt```

This file is stored in:

* The **container layer (read-write)**

It does NOT modify the image

---

# Copy-on-Write (CoW)

## What is Copy-on-Write?

Copy-on-Write is a mechanism where:

* **Read-only layers are not modified**
* Changes are written to a new layer

---

## How It Works

### Case 1: Read File

* File exists in image layer
* Container reads it directly

---

### Case 2: Modify File

* Original file is in **read-only layer**
* Docker:

  1. Copies file to container layer
  2. Modifies it there

This is Copy-on-Write

---

## Summary

| Layer           | Type       | Behavior       |
| --------------- | ---------- | -------------- |
| Image Layer     | Read-only  | Shared         |
| Container Layer | Read-write | Stores changes |

---

# Volumes

## What are Volumes?

Volumes are used to **persist data outside the container lifecycle**.

Data survives:

* Container restart
* Container deletion

---

## Create Volume

```docker volume create data_volume```

Stored under:

```/var/lib/docker/volumes/```

---

## Volume Mounting

### Old Syntax

```docker run -v data_volume:/var/lib/mysql mysql```

Mounts:

* Volume → Container path

---

### What Happens

* Data written to `/var/lib/mysql` goes to the volume
* Data persists even if container is removed

---

# Bind Mounting

## What is Bind Mounting?

Bind mounting links a **host directory directly to a container**.

---

## New Syntax (Recommended)

```docker run --mount type=bind,source=/data/mysql,target=/var/lib/mysql mysql```

---

## Difference: Volume vs Bind Mount

| Feature     | Volume            | Bind Mount      |
| ----------- | ----------------- | --------------- |
| Location    | Managed by Docker | Host filesystem |
| Portability | High              | Low             |
| Control     | Docker-managed    | User-managed    |
| Use Case    | Production        | Development     |

---

## Example: Bind Mount

```docker run -v /host/data:/container/data```

Direct mapping:

* Host → Container

---

# Volume Mounting vs Bind Mounting

## Volume Mounting

* Managed by Docker
* Safer and portable
* Ideal for databases

---

## Bind Mounting

* Uses host filesystem
* More control
* Useful for development and debugging

---

# Key Takeaways

* Docker uses **layered filesystem architecture**
* Image layers are **read-only**
* Container layer is **read-write**
* Copy-on-Write avoids modifying base image
* Volumes persist data
* Bind mounts map host directories directly

---

# Pro Tip

* Use **volumes** for production data
* Use **bind mounts** for local development
* Prefer **overlay2** storage driver for best performance










# Docker Volume Drivers & Kubernetes Storage (Detailed)

---

# Docker Volume Drivers

## What is the Local Volume Plugin?

The **local volume plugin** is the default Docker volume driver.

It:
- Creates volumes on the Docker host  
- Stores data under: `/var/lib/docker/volumes/`

This is managed entirely by Docker (not directly by the user)

### Common Volume Drivers

Docker supports external storage systems via volume drivers:

* Local
* Azure File Storage
* Convoy
* DigitalOcean Block Storage
* Flocker
* gce-docker
* GlusterFS
* NetApp
* RexRay
* Portworx
* VMware vSphere Storage

### Using External Volume Drivers

Example with AWS EBS using rexray:
```
docker run -it \
  --name mysql \
  --volume-driver rexray/ebs \
  --mount src=ebs-vol,target=/var/lib/mysql \
  mysql
```

What happens:
* Docker uses RexRay plugin
* Provisions an EBS volume
* Mounts it to `/var/lib/mysql` inside container




## Volumes and Mounts
### Example: Pod with Volume
```
apiVersion: v1
kind: Pod
metadata:
    name: random-number-generator
spec:
    containers:
    - image: alpine
      name: alpine
      command: ["/bin/sh", "-c"]
      args: ["shuf -i 0-100 -n 1 >> /opt/number.out;"]
      volumeMounts:
      - mountPath: /opt
        name: data-volume
    volumes:
    - name: data-volume
      hostPath:
        path: /data
        type: Directory
```

#### How This Works
`volumeMounts` → mounts volume inside container
`volumes` → defines the storage source

In this case:

`/data` (host) → mounted to `/opt` (container)

#### Important Note: hostPath
* Ties Pod to a specific node
* Not portable across cluster
* Not recommended for production

### Using Cloud Storage (EBS Example)
*we replace 'hostpath' field of the volume with the:*

```
volumes:
- name: data-volume
  awsElasticBlockStore:
    volumeID: {volume-id}
    fsType: ext4
```
Now:
* Storage is external (AWS EBS)
* Not tied to a specific node

## Persistent Volumes
What is a Persistent Volume?

A PersistentVolume (PV) is:
* A cluster-wide storage resource
* Created and managed by administrators

### Example
```
apiVersion: v1
kind: PersistentVolume
metadata:
    name: pv-vol1
spec:
    accessModes:
    - ReadWriteOnce
    capacity:
      storage: 1Gi
    hostPath:
      path: /tmp/data
```

The access mode defines how a volume should be mounted on the hosts, whether in:
* ReadWriteOnce (RWO) → mounted by one node
* ReadOnlyMany (ROX) → read-only by many nodes
* ReadWriteMany (RWX) → read-write by many nodes

If you use one of the supported storage solutions such as AWS:
```
apiVersion: v1
kind: PersistentVolume
metadata:
    name: pv-vol1
spec:
    accessModes:
    - ReadWriteOnce
    capacity:
      storage: 1Gi
    awsElasticBlockStore:
      volumeID: {volume-id}
      fsType: ext4
```  

Decouples storage from nodes


### Commands
```kubectl create -f pv-definition.yaml```

```kubectl get persistentvolume```
or
```kubectl get pvc```



## Persistent Volume Claims

What is a PVC?

A PersistentVolumeClaim (PVC) is:
* A request for storage by users
* Bound to a matching PV

Every persistent volume claim is bound to a single persistent volume.

If there are many possible matches for a single claim and you would like to specifically use a particular volume, you could still use labels and selectors to bind the right volumes.

Matching Specific PV:
```
labels:
    name: my-pv
```

and

```
selector:
    matchLabels:
        name: my-pv
```

#### Binding Process

Kubernetes matches:
* Requested size
* Access mode
* Labels/selectors


#### Example - Definition
```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
    name: myclaim
spec:
    accessModes:
    - ReadWriteOnce
    resources:
      requests:
        storage: 500Mi
```


### Reclaim Policies
Defines what happens when PVC is deleted (using the field `persistentVolumeReclaimPolicy`):
1. `Retain` (default)
    * PV remains
    * Manual cleanup required
2. `Delete`
    * PV and storage deleted automatically
3. `Recycle` (deprecated)
    * Data wiped and reused

Modern approach: use **StorageClass + dynamic provisioning**




### Commands
```kubectl create -f pvc-definition.yaml```

```kubectl get persistentvolumeclaim```
or 
```kubectl get pvc```

```kubectl delete persistentVolume myclaim```


### Using PVCs in Pods
Once you create a PVC use it in a POD definition file by specifying the PVC Claim name under persistentVolumeClaim section in the volumes section like this:
```
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
    - name: myfrontend
      image: nginx
      volumeMounts:
      - mountPath: "/var/www/html"
        name: mypd
  volumes:
    - name: mypd
      persistentVolumeClaim:
        claimName: myclaim
```

### How It Works
* PVC → requests storage
* PV → provides storage
* Pod → consumes storage

### Works With
* Pods
* ReplicaSets
* Deployments

Must be defined in the **pod template**


## Command Explanation
```k exec webapp -- cat /log/app.log```

### Breakdown
* `exec` → run command inside a container
* `webapp` → Pod name
* `--` → separates kubectl command from container command
* `cat /log/app.log` → command executed inside container

### What It Does

Executes inside the container:
```cat /log/app.log```

Result:
* Prints contents of /log/app.log from inside the Pod

### Use Cases
* Debugging applications
* Checking logs
* Verifying file contents inside containers


## Key Takeaways
* Docker uses volume drivers for storage integration
* Kubernetes abstracts storage using PV and PVC
* PV = supply, PVC = demand
* Volumes enable persistent storage for Pods



## Scenarios
### Scenario 1
Configure a volume to store these logs at /var/log/webapp on the host.
Use the spec provided below.

* Name: webapp
* Image Name: kodekloud/event-simulator
* Volume HostPath: /var/log/webapp
* Volume Mount: /log

### Solution:
First delete the existing pod by running the following command: 
`kubectl delete po webapp`

then use the below manifest file to create a webapp pod with given properties as follows:
```
apiVersion: v1
kind: Pod
metadata:
  name: webapp
spec:
  containers:
  - name: event-simulator
    image: kodekloud/event-simulator
    env:
    - name: LOG_HANDLERS
      value: file
    volumeMounts:
    - mountPath: /log
      name: log-volume

  volumes:
  - name: log-volume
    hostPath:
      # directory location on host
      path: /var/log/webapp
      # this field is optional
      type: Directory
```

Then run the command ```kubectl create -f <file-name>.yaml``` to create a pod.

---

### Scenario 2
Create a Persistent Volume with the given specification:
* Volume Name: pv-log
* Storage: 100Mi
* Access Modes: ReadWriteMany
* Host Path: /pv/log
* Reclaim Policy: Retain

### Solution
Use the following manifest file to create a pv-log persistent volume:
```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-log
spec:
  persistentVolumeReclaimPolicy: Retain
  accessModes:
    - ReadWriteMany
  capacity:
    storage: 100Mi
  hostPath:
    path: /pv/log
```
Then run the command `kubectl create -f <file-name>.yaml` to create a PV from manifest file.

---

### Scenario 3
Let us claim some of that storage for our application. Create a Persistent Volume Claim with the given specification:
* Persistent Volume Claim: claim-log-1
* Storage Request: 50Mi
* Access Modes: ReadWriteOnce

### Solution
Solution manifest file to create a claim-log-1 PVC with given properties as follows:
```
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: claim-log-1
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 50Mi
```
Then run `kubectl create -f <file-name>.yaml` to create a PVC from the manifest file.

---

### Scenario 4
Update the Access Mode on the claim to bind it to the PV.
Delete and recreate the claim-log-1.

### Solution
To delete the existing pvc:

`kubectl delete pvc claim-log-1`

Solution manifest file to create a claim-log-1 PVC with correct Access Modes as follows:
```
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: claim-log-1
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 50Mi
```
Then run `kubectl create -f <file-name>.yaml`

---

### Scenario 5
Update the webapp pod to use the persistent volume claim as its storage.
Replace hostPath configured earlier with the newly created PersistentVolumeClaim:
* Name: webapp
* Image Name: kodekloud/event-simulator
* Volume: PersistentVolumeClaim=claim-log-1
* Volume Mount: /log

### Solution
To delete the webapp pod first:

`kubectl delete po webapp`
Add `--force` flag in above command, if you would like to delete the pod without any delay.

To create a new webapp pod with given properties as follows:
```
apiVersion: v1
kind: Pod
metadata:
  name: webapp
spec:
  containers:
  - name: event-simulator
    image: kodekloud/event-simulator
    env:
    - name: LOG_HANDLERS
      value: file
    volumeMounts:
    - mountPath: /log
      name: log-volume

  volumes:
  - name: log-volume
    persistentVolumeClaim:
      claimName: claim-log-1
```
Then run the command `kubectl create -f <file-name>.yaml` to create a pod from the definition file.