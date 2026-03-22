This YAML defines a Deployment that manages and maintains 3 Pods running an nginx container.
The Deployment myapp-deployment ensures 3 replicas are always running and handles updates/rollbacks automatically.
It uses the Pod template to create Pods labeled type: front-end, which are managed through an underlying ReplicaSet.
```
apiVersion: apps/v1 
kind: Deployment 
metadata:
    name: myapp-deployment
    labels:
        app: myapp
        type: front-end
spec:
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
    replicas: 3
    selector:
        matchLabels:
            type: front-end
```


## Commands
### Create a Deployment in the cluster using the configuration defined in the YAML file:
```kubectl create -f deployment-definition.yaml```
### List all Deployments in the current namespace along with their status:
```kubectl get deployments```
### List all common Kubernetes resources in the current namespace, such as Pods, Services, Deployments, and ReplicaSets:
```kubectl get all```
### Create a Deployment with the specified name, container image, and number of replicas. It automatically generates the underlying ReplicaSet and Pods based on these parameters:
```kubectl create deployment {name} --image={image-name} --replicas={number-of-replicas}```