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

### To create a Pod using the configuration file:
```kubectl create -f pod-definition.yaml```

### Get all Pods:
```kubectl get pods```

### List all Pods with additional details beyond the default output. It includes info like the Pod’s IP address, node it’s running on:
```kubectl get pods -o wide```

### Get detailed information about a specific Pod:
```kubectl describe pod {pod-name}```

### Extract a pod definition to a file (if you aren't given a pod definition file):
```kubectl get pod {pod-name} -o yaml > pod-definition.yaml``

### Modify properties of a pod:
```kubectl edit pod {pod-name}```
#### Only the properties listed below are editable:
- spec.containers[*].image
- spec.initContainers[*].image
- spec.activeDeadlineSeconds
- spec.tolerations
- spec.terminationGracePeriodSeconds

### Delete a pod
```kubectl delete pod {pod-name}```

### Create or update Kubernetes resources defined in the {filename}.yaml file. It ensures the cluster state matches the configuration in that file.
```kubectl apply -f pod-definition.yaml ```
