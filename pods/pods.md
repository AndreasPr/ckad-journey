## Every Kubernetes resource definition (YAML file) follows a consistent structure with four top-level fields:
- apiVersion  <!-- Defines which version of the Kubernetes API you're using. -->
- kind  <!--Specifies the type of Kubernetes object.-->
- metadata <!--Provides identifying information about the object. -->
- spec <!--Defines the desired state of the object.-->


## Instead of using commands, Kubernetes encourages a declarative approach using YAML files. Example: pod-definition.yaml
```
apiVersion: v1
kind: Pod
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


## Commands
## Create a Pod in Kubernetes is by using the command:
```kubectl run nginx --image=nginx```

## Generate the YAML manifest for a Pod using an image without actually creating it. It outputs the configuration in YAML format so you can inspect or save it before applying:
```kubectl run nginx --image=nginx --dry-run=client -o yaml```

## If you want to save the output to a file named {filename}.yaml. This lets you edit or reuse the configuration before applying it to the cluster:
```kubectl run nginx --image=nginx --dry-run=client -o yaml > pod-definition.yaml```

## Create a Pod named myapp using the andreas/myapp image. The -- --color green part passes --color green as command-line arguments to the container at runtime:
```kubectl run myapp --image=andreas/myapp -- --color green```

### Interesting comment

If you add `--command`, it tells Kubernetes to treat the arguments after `--` as the **actual command (ENTRYPOINT)** instead of just arguments.

*Without* `--command`:

* `--color green` is passed as **arguments** to the container’s default command.

*With* `--command`:

```kubectl run myapp --image=andreas/myapp --command -- --color green```

* `--color green` **replaces the container’s default command entirely** and is executed as the main command.


### Create a Pod using the configuration file:
```kubectl create -f pod-definition.yaml```

### Get all Pods:
```kubectl get pods```

### List all Pods with additional details beyond the default output. It includes info like the Pod’s IP address, node it’s running on:
```kubectl get pods -o wide```

### Get detailed information about a specific Pod:
```kubectl describe pod {pod-name}```

### Extract a pod definition to a file (if you aren't given a pod definition file):
```kubectl get pod {pod-name} -o yaml > pod-definition.yaml```


### Count Pods with a Specific Label
```kubectl get pods --selector env=dev --no-headers | wc -l```

What it does:

* Filters Pods with label `env=dev`
* `--no-headers removes the header row
* `wc -l` counts the number of matching Pods

**Result:** Total number of Pods in the dev environment

### Get All Resources with a Label
```kubectl get all --selector env=prod --no-headers```

What it does:

* Retrieves all common resources (Pods, Services, Deployments, etc.)
* Filters by label env=prod
* Hides headers for cleaner output

Useful for:

* Viewing everything related to a specific environment


### Filter with Multiple Labels (AND condition)
```kubectl get all --selector env=prod,bu=finance,tier=finance```

What it does:

Selects resources that match ALL of the following:
* `env=prod`
* `bu=finance`
* `tier=finance`

#### Important:

* Comma-separated labels = logical AND
* Resource must match every label


### Edit properties of a pod:
```kubectl edit pod {pod-name}```
#### In Kubernetes, you cannot modify most fields of an existing Pod. Only the following fields are editable:
- spec.containers[*].image
- spec.initContainers[*].image
- spec.activeDeadlineSeconds
- spec.tolerations
- spec.terminationGracePeriodSeconds

Fields such as environment variables, service accounts, and resource limits cannot be changed once the Pod is running.

What if you need to change non-editable fields?

**You have 2 main options:**

**Option 1**: Use ```kubectl edit``` (and recreate the Pod)
```kubectl edit pod <pod-name>```

* This opens the Pod definition in an editor (e.g., `vi`)
* Make your desired changes and try to save

You will get an error if you modify non-editable fields.

However:

* A **temporary file** containing your changes will be saved (path shown in the error message)

Then:

1. Delete the existing Pod:

```kubectl delete pod webapp```

2. Recreate the Pod using the saved file:

```kubectl create -f /tmp/kubectl-edit-xxxx.yaml```

---

**Option 2**: Export, Modify, and Recreate

1. Export the current Pod definition:

```kubectl get pod webapp -o yaml > my-new-pod.yaml```

2. Edit the file:

```vi my-new-pod.yaml```

3. Delete the existing Pod:

```kubectl delete pod webapp```

4. Create a new Pod with the updated configuration:

```kubectl create -f my-new-pod.yaml```


### Delete a pod
```kubectl delete pod {pod-name}```

### Create or update Kubernetes resources defined in the {filename}.yaml file. It ensures the cluster state matches the configuration in that file.
```kubectl apply -f pod-definition.yaml ```

# Scenarios - Tasks

## Task 1: Create Pods with Labels

Create three Pods with the following specifications:
- Names: nginx1, nginx2, nginx3
- Image: nginx
- Label: app=v1

### Solution
```bash
kubectl run nginx1 --image=nginx --restart=Never --labels=app=v1
kubectl run nginx2 --image=nginx --restart=Never --labels=app=v1
kubectl run nginx3 --image=nginx --restart=Never --labels=app=v1
```

Alternatively:
```bash
for i in `seq 1 3`; do kubectl run nginx$i --image=nginx -l app=v1; done
```

## Task 2: Show Labels of All Pods

### Solution

```bash
kubectl get pods --show-labels
```

---

## Task 3: Update Label of a Pod

Change the label of pod nginx2 to app=v2.

### Solution

```bash
kubectl label pod nginx2 app=v2 --overwrite
```

---

## Task 4: Display Pods with Label Column

Show all Pods with a column displaying the app label.

### Solution

```bash
kubectl get pods -L app
```

---

## Task 5: Filter Pods by Label

Get only Pods with label app=v2.

### Solution

```bash
kubectl get pods -l app=v2
```

---

## Task 6: Filter Pods with Multiple Conditions

Get Pods with app=v2 and not tier=frontend.

### Solution

```bash
kubectl get pods -l app=v2,tier!=frontend
```

---

## Task 7: Add Label to Multiple Pods

Add label tier=web to all Pods with app=v1 or app=v2.

### Solution

```bash
kubectl label pods -l "app in (v1,v2)" tier=web
```

---

## Task 8: Add Annotation

Add annotation owner=marketing to all Pods with app=v2.

### Solution

```bash
kubectl annotate pods -l app=v2 owner=marketing
```

---

## Task 9: Remove Label

Remove the label app from all three Pods.

### Solution

```bash
kubectl label pods nginx1 nginx2 nginx3 app-
```

---

## Task 10: Add Annotation to Specific Pods

Annotate nginx1, nginx2, nginx3 with description="my description".

### Solution

```bash
kubectl annotate pods nginx1 nginx2 nginx3 description="my description"
```

---

## Task 11: Check Annotations

Check annotations for pod nginx1.

### Solution

```bash
kubectl describe pod nginx1
```

---

## Task 12: Remove Annotations

Remove annotations description and owner from the Pods.

### Solution

```bash
kubectl annotate pods nginx1 nginx2 nginx3 description- owner-
```

---

## Task 13: Cleanup

Delete the created Pods.

### Solution

```bash
kubectl delete pods nginx1 nginx2 nginx3
```

---

# Pod Placement

## Task 14: Schedule Pod Using Node Label

Create a Pod that runs on a node with label:
accelerator=nvidia-tesla-p100

### Solution

Label a node:

```bash
kubectl label nodes <node-name> accelerator=nvidia-tesla-p100
```

Pod YAML:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: cuda-test
spec:
  containers:
  - name: cuda-test
    image: k8s.gcr.io/cuda-vector-add:v0.1
  nodeSelector:
    accelerator: nvidia-tesla-p100
```

---

## Task 15: Use Node Affinity

### Solution

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: affinity-pod
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: accelerator
            operator: In
            values:
            - nvidia-tesla-p100
  containers:
  - name: nginx
    image: nginx
```

---

## Task 16: Schedule Pod to Specific Node

Create a Pod that runs on node node01 using nodeName.

### Solution

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nodename-pod
spec:
  nodeName: node01
  containers:
  - name: nginx
    image: nginx
```

---

## Task 17: Taints and Tolerations

Taint a node and create a Pod that tolerates it.

### Solution

Taint node:

```bash
kubectl taint node node1 tier=frontend:NoSchedule
```

Pod YAML:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: frontend
spec:
  containers:
  - name: nginx
    image: nginx
  tolerations:
  - key: "tier"
    operator: "Equal"
    value: "frontend"
    effect: "NoSchedule"
```

---

## Task 18: Schedule Pod on Control Plane Node

Create a Pod that runs on node controlplane using nodeSelector and tolerations.

### Solution

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: frontend
spec:
  containers:
  - name: nginx
    image: nginx
  nodeSelector:
    kubernetes.io/hostname: controlplane
  tolerations:
  - key: "node-role.kubernetes.io/control-plane"
    operator: "Exists"
    effect: "NoSchedule"
```

Apply:

```bash
kubectl apply -f pod.yaml
```
