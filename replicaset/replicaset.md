This YAML defines a ReplicaSet that ensures 6 identical Pods are always running.
It creates a ReplicaSet named myapp-replicaset that manages Pods with the label type: front-end.
The Pods it creates run a container named nginx-container using the nginx image, based on the template defined under spec.template.
```
apiVersion: apps/v1 
kind: ReplicaSet 
metadata:
    name: myapp-replicaset
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
    replicas: 6
    selector:
        matchLabels:
            type: front-end
```


## Commands
### Create a new ReplicaSet from the specified YAML file:
```kubectl create -f replicaset-definition.yaml```

### List all ReplicaSets in the current namespace:
```kubectl get replicaset```
or 
```kubectl get rs```

### Show detailed information about the ReplicaSet, including its Pods, events, labels, and current status:
```kubectl describe replicaset new-replica-set```

### Display documentation for the ReplicaSet resource, explaining its fields and structure in Kubernetes YAML:
```kubectl explain replicaset```

### Delete the specified ReplicaSet and its managed Pods:
```kubectl delete replicaset myapp-replicaset```

### Edit the live ReplicaSet configuration in your default editor so you can modify it directly in the cluster:
```kubectl edit rs new-replica-set```
### Save your changes and exit:
```:wq!```

### Scaling Options:
#### 1: Update the existing ReplicaSet by replacing it with the configuration in the file:
```kubectl replace -f replicaset-definition.yaml```

#### 2: Change the number of desired Pods in the ReplicaSet to 6:
```kubectl scale --replicas=6 -f replicaset-definition.yaml```

#### 3: Update the existing ReplicaSet to run 6 Pods. It changes the desired replica count without modifying the original YAML file:
```kubectl scale --replicas=6 rs myapp-replicaset```

