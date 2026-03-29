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
