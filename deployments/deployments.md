# Deployment in Kubernetes

A **Deployment** is a higher-level Kubernetes resource that manages **ReplicaSets** and ensures a specified number of Pods are running.

It provides additional capabilities such as:

* Rolling updates
* Rollbacks to previous versions
* Declarative updates
* Self-healing (via underlying ReplicaSets)

---

## Example Definition

The following YAML defines a Deployment that maintains **3 Pods** running an `nginx` container:

```yaml
apiVersion: apps/v1 
kind: Deployment 
metadata:
  name: myapp-deployment
  labels:
    app: myapp
    type: front-end
spec:
  replicas: 3
  selector:
    matchLabels:
      type: front-end
  template: 
    metadata:
      name: myapp-pod
      labels:
        app: myapp
        type: front-end
    spec:
      containers:
      - name: nginx-container
        image: nginx
```

---

## Explanation

* **metadata.name**: Name of the Deployment
* **spec.replicas**: Desired number of Pods (3)
* **spec.selector**: Identifies which Pods belong to this Deployment
* **spec.template**: Blueprint for creating Pods

  * Must match the selector labels
* **containers**: Defines the container(s) running inside each Pod

### How it works internally

* The Deployment creates a **ReplicaSet**
* The ReplicaSet creates and manages the Pods
* The Deployment manages the lifecycle of the ReplicaSet (updates, rollbacks)

---

# Common Commands

## Create

Create a Deployment from a YAML file:

```bash
kubectl create -f deployment-definition.yaml
```

---

## Get

List all Deployments:

```bash
kubectl get deployments
```

List all common Kubernetes resources:

```bash
kubectl get all
```

---

## Imperative Creation

Create a Deployment directly from the command line:

```bash
kubectl create deployment {name} --image={image-name} --replicas={number-of-replicas}
```

Example:

```bash
kubectl create deployment myapp --image=nginx --replicas=3
```

---

# Key Features of Deployments

## 1. Rolling Updates

When you update the Deployment (e.g., change image version), Kubernetes:

* Gradually replaces old Pods with new ones
* Ensures zero downtime (by default)

---

## 2. Rollbacks

You can revert to a previous version if something goes wrong:

```bash
kubectl rollout undo deployment myapp-deployment
```

---

## 3. Scaling

Scale the number of Pods:

```bash
kubectl scale deployment myapp-deployment --replicas=5
```

---

## 4. Status and History

Check rollout status:

```bash
kubectl rollout status deployment myapp-deployment
```

View rollout history:

```bash
kubectl rollout history deployment myapp-deployment
```

---

# Important Notes

* The **selector must match the Pod template labels**, otherwise the Deployment will not manage the Pods correctly.
* Deployments are the **recommended way** to manage stateless applications in Kubernetes.
* You should rarely create ReplicaSets directly—Deployments handle them for you.

# Scenarios

## Task 1: Create a Deployment

Create a Deployment with the following specifications:
- Name: nginx
- Image: nginx:1.18.0
- Replicas: 2
- Container port: 80

### Solution
```bash
kubectl create deployment nginx --image=nginx:1.18.0 --replicas=2 --port=80
````

---

## Task 2: View Deployment YAML

### Solution

```bash
kubectl get deployment nginx -o yaml
```

---

## Task 3: View ReplicaSet YAML

### Solution

```bash
kubectl get rs
kubectl get rs <replicaset-name> -o yaml
```

---

## Task 4: View Pod YAML

### Solution

```bash
kubectl get pods
kubectl get pod <pod-name> -o yaml
```

---

## Task 5: Check Rollout Status

### Solution

```bash
kubectl rollout status deployment nginx
```

---

## Task 6: Update Image

Update the container image to nginx:1.19.8.

### Solution

```bash
kubectl set image deployment nginx nginx=nginx:1.19.8
```

---

## Task 7: Check Rollout History

### Solution

```bash
kubectl rollout history deployment nginx
kubectl get deployment nginx
kubectl get rs
kubectl get pods
```

---

## Task 8: Rollback Deployment

Undo the latest rollout and verify the image.

### Solution

```bash
kubectl rollout undo deployment nginx
kubectl get pods
kubectl describe pod <pod-name>
```

---

## Task 9: Perform Failed Update

Update the image to an invalid version nginx:1.91.

### Solution

```bash
kubectl set image deployment nginx nginx=nginx:1.91
kubectl rollout status deployment nginx
kubectl get pods
```

---

## Task 10: Rollback to Specific Revision

Rollback to revision 2.

### Solution

```bash
kubectl rollout undo deployment nginx --to-revision=2
kubectl rollout status deployment nginx
```

---

## Task 11: Check Revision Details

### Solution

```bash
kubectl rollout history deployment nginx --revision=4
```

---

## Task 12: Scale Deployment

Scale the deployment to 5 replicas.

### Solution

```bash
kubectl scale deployment nginx --replicas=5
kubectl get pods
```

---

## Task 13: Configure Autoscaling

Autoscale between 5 and 10 replicas with CPU utilization at 80 percent.

### Solution

```bash
kubectl autoscale deployment nginx --min=5 --max=10 --cpu=80%
kubectl get hpa
```

---

## Task 14: Pause Rollout

### Solution

```bash
kubectl rollout pause deployment nginx
```

---

## Task 15: Update Image While Paused

Update image to nginx:1.19.9 and verify no rollout occurs.

### Solution

```bash
kubectl set image deployment nginx nginx=nginx:1.19.9
kubectl rollout history deployment nginx
```

---

## Task 16: Resume Rollout

### Solution

```bash
kubectl rollout resume deployment nginx
kubectl rollout history deployment nginx
```

---

## Task 17: Cleanup

Delete deployment and autoscaler.

### Solution

```bash
kubectl delete deployment nginx
kubectl delete hpa nginx
```

---

# Canary Deployment

## Task 18: Deploy Version v1

Deploy 3 replicas of version v1.

### Solution

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app-v1
  labels:
    app: my-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
      version: v1
  template:
    metadata:
      labels:
        app: my-app
        version: v1
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
        volumeMounts:
        - name: workdir
          mountPath: /usr/share/nginx/html
      initContainers:
      - name: install
        image: busybox:1.28
        command:
        - /bin/sh
        - -c
        - echo version-1 > /work-dir/index.html
        volumeMounts:
        - name: workdir
          mountPath: /work-dir
      volumes:
      - name: workdir
        emptyDir: {}
```

---

## Task 19: Create Service

### Solution

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app-svc
spec:
  selector:
    app: my-app
  ports:
  - port: 80
    targetPort: 80
```

---

## Task 20: Test Service

### Solution

```bash
kubectl run -it --rm --restart=Never busybox --image=busybox -- wget -qO- my-app-svc
```

---

## Task 21: Deploy Version v2

Deploy 1 replica of version v2.

### Solution

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app-v2
  labels:
    app: my-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-app
      version: v2
  template:
    metadata:
      labels:
        app: my-app
        version: v2
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
        volumeMounts:
        - name: workdir
          mountPath: /usr/share/nginx/html
      initContainers:
      - name: install
        image: busybox:1.28
        command:
        - /bin/sh
        - -c
        - echo version-2 > /work-dir/index.html
        volumeMounts:
        - name: workdir
          mountPath: /work-dir
      volumes:
      - name: workdir
        emptyDir: {}
```

---

## Task 22: Verify Load Balancing

### Solution

```bash
kubectl run -it --rm --restart=Never busybox --image=busybox -- /bin/sh -c "while sleep 1; do wget -qO- my-app-svc; done"
```

---

## Task 23: Promote v2 and Remove v1

### Solution

```bash
kubectl scale deployment my-app-v2 --replicas=4
kubectl delete deployment my-app-v1
```
