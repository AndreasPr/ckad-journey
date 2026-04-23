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

```bash
docker build Dockerfile -t andreas/my-app
```

Creates layered image:

* Base image
* Installed packages
* Application code

---

# What Happens When You Modify a Container?

If you run a container and create a file:

```bash
touch temp.txt
```

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

```bash
docker volume create data_volume
```

Stored under:

```/var/lib/docker/volumes/```

---

## Volume Mounting

### Old Syntax

```bash
docker run -v data_volume:/var/lib/mysql mysql
```

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

```bash 
docker run --mount type=bind,source=/data/mysql,target=/var/lib/mysql mysql
```

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

```bash
docker run -v /host/data:/container/data
```

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






# Docker Volume Drivers & Kubernetes Storage (More Info)

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

```bash
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
```yaml
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

### Using Cloud Storage (AWS EBS Example)
*we replace 'hostpath' field of the volume with the:*

```yaml
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
```yaml
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
* `ReadWriteOnce` (RWO) → mounted by one node
* `ReadOnlyMany` (ROX) → read-only by many nodes
* `ReadWriteMany` (RWX) → read-write by many nodes

If you use one of the supported storage solutions such as AWS:
```yaml
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

**Decouples storage from nodes**


### Commands
```kubectl create -f pv-definition.yaml```

```kubectl get persistentvolume```

or

```kubectl get pv```



## Persistent Volume Claims

What is a PVC?

A PersistentVolumeClaim (PVC) is:
* A request for storage by users
* Bound to a matching PV

Every persistent volume claim is bound to a single persistent volume.

If there are many possible matches for a single claim and you would like to specifically use a particular volume, you could still use labels and selectors to bind the right volumes.

Matching Specific PV:
```yaml
labels:
    name: my-pv
```

and

```yaml
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
```yaml
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
```yaml
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
        name: mypod
  volumes:
    - name: mypod
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



# Scenarios
## Scenario 1
Configure a volume to store these logs at `/var/log/webapp` on the host.
Use the spec provided below.

* Name: `webapp`
* Image Name: `andreas/event-simulator`
* Volume HostPath: `/var/log/webapp`
* Volume Mount: `/log`

## Solution:
First delete the existing pod by running the following command: 
`kubectl delete pod webapp`

then use the below manifest file to create a `webapp` pod with given properties as follows:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapp
spec:
  containers:
  - name: event-simulator
    image: andreas/event-simulator
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

## Scenario 2
Create a Persistent Volume with the given specification:
* Volume Name: `pv-log`
* Storage: `100Mi`
* Access Modes: `ReadWriteMany`
* Host Path: `/pv/log`
* Reclaim Policy: `Retain`

## Solution
Use the following manifest file to create a pv-log persistent volume:
```yaml
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

## Scenario 3
Let us claim some of that storage for our application. Create a Persistent Volume Claim with the given specification:
* Persistent Volume Claim: `claim-log-1`
* Storage Request: `50Mi`
* Access Modes: `ReadWriteOnce`

## Solution
Solution manifest file to create a `claim-log-1` PVC with given properties as follows:
```yaml
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

## Scenario 4
Update the Access Mode on the claim to bind it to the PV.
Delete and recreate the `claim-log-1`.

## Solution
To delete the existing pvc:

`kubectl delete pvc claim-log-1`

Solution manifest file to create a `claim-log-1` PVC with correct Access Modes as follows:
```yaml
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

## Scenario 5
Update the `webapp` pod to use the persistent volume claim as its storage.
Replace `hostPath` configured earlier with the newly created PersistentVolumeClaim:
* Name: `webapp`
* Image Name: `andreas/event-simulator`
* Volume: `PersistentVolumeClaim=claim-log-1`
* Volume Mount: `/log`

## Solution
To delete the `webapp` pod first:

`kubectl delete pod webapp`

Add `--force` flag in above command, if you would like to delete the pod without any delay.

To create a new `webapp` pod with given properties as follows:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapp
spec:
  containers:
  - name: event-simulator
    image: andreas/event-simulator
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


---

## Scenario 6: Shared Volume with emptyDir (Multi-Container Pod)

Create a Pod named busybox with two containers:
- Both use the busybox image
- Both run sleep 3600
- Both mount a shared emptyDir volume at /etc/foo

Then:
1. Connect to container busybox2 and extract the first column of /etc/passwd into /etc/foo/passwd
2. Connect to container busybox and print the contents of /etc/foo/passwd
3. Delete the Pod

### Key Concepts
- emptyDir is a temporary volume shared between containers in the same Pod
- Data persists only for the lifetime of the Pod
- Useful for sidecar and multi-container communication

### Solution
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: busybox
spec:
  containers:
  - name: busybox
    image: busybox
    command: ["/bin/sh", "-c", "sleep 3600"]
    volumeMounts:
    - name: myvolume
      mountPath: /etc/foo
  - name: busybox2
    image: busybox
    command: ["/bin/sh", "-c", "sleep 3600"]
    volumeMounts:
    - name: myvolume
      mountPath: /etc/foo
  volumes:
  - name: myvolume
    emptyDir: {}
````

```bash
kubectl apply -f pod.yaml

kubectl exec -it busybox -c busybox2 -- /bin/sh
cat /etc/passwd | cut -d ':' -f 1 > /etc/foo/passwd
exit

kubectl exec -it busybox -c busybox -- cat /etc/foo/passwd

kubectl delete pod busybox
```

---

## Task 2: Create PersistentVolume

Create a PersistentVolume named myvolume:

* Size: 10Gi
* AccessModes: ReadWriteOnce, ReadWriteMany
* storageClassName: normal
* hostPath: /etc/foo

### Key Concepts

* PersistentVolume is cluster-wide storage
* hostPath ties storage to a specific node (not suitable for multi-node clusters)
* AccessModes define how Pods can use the volume

### Solution

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: myvolume
spec:
  storageClassName: normal
  capacity:
    storage: 10Gi
  accessModes:
  - ReadWriteOnce
  - ReadWriteMany
  hostPath:
    path: /etc/foo
```

```bash
kubectl apply -f pv.yaml
kubectl get pv
```

---

## Task 3: Create PersistentVolumeClaim

Create a PersistentVolumeClaim named mypvc:

* Request: 4Gi
* AccessMode: ReadWriteOnce
* storageClassName: normal

### Key Concepts

* PVC requests storage from available PVs
* Binding occurs when requirements match
* Once bound, PVC is tied to a specific PV

### Solution

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mypvc
spec:
  storageClassName: normal
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 4Gi
```

```bash
kubectl apply -f pvc.yaml
kubectl get pvc
kubectl get pv
```

---

## Task 4: Use PVC in a Pod

Create a Pod that:

* Uses busybox image
* Runs sleep 3600
* Mounts PVC at /etc/foo

Then:

* Copy /etc/passwd into /etc/foo/passwd

### Key Concepts

* PVC abstracts storage from Pods
* Pods consume storage via PVC, not PV directly

### Solution

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: busybox
spec:
  containers:
  - name: busybox
    image: busybox
    command: ["/bin/sh", "-c", "sleep 3600"]
    volumeMounts:
    - name: myvolume
      mountPath: /etc/foo
  volumes:
  - name: myvolume
    persistentVolumeClaim:
      claimName: mypvc
```

```bash
kubectl apply -f pod.yaml
kubectl exec busybox -- cp /etc/passwd /etc/foo/passwd
```

---

## Task 5: Verify Data Persistence Across Pods

Create a second Pod (busybox2) using the same PVC.

Verify:

* /etc/foo/passwd exists in the second Pod

### Key Concepts

* Data persists beyond Pod lifecycle
* hostPath limitation: data only visible if Pods run on same node
* In multi-node clusters, use network storage (e.g. NFS, EBS, etc.)

### Solution

```bash
# duplicate pod.yaml and change name to busybox2
kubectl apply -f pod.yaml

kubectl exec busybox2 -- ls /etc/foo
```

Cleanup:

```bash
kubectl delete pod busybox busybox2
kubectl delete pvc mypvc
kubectl delete pv myvolume
```

---

## Task 6: Copy File from Pod to Local Machine

Create a Pod and copy /etc/passwd to your local system.

### Key Concepts

* kubectl cp allows file transfer between local machine and Pods
* Useful for debugging and data extraction

### Solution

```bash
kubectl run busybox --image=busybox --restart=Never -- sleep 3600

kubectl cp busybox:/etc/passwd ./passwd

cat passwd
```

