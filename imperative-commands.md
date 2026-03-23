In Kubernetes, we will mostly work in a declarative manner using YAML configuration files. However, imperative commands are extremely useful when we need to act quickly, whether it’s creating a resource on the spot or generating a template we can refine later. 

Before using these commands, we should understand two key options:
1. --dry-run=client
Prevents the resource from being created. Instead, it validates our command and shows whether it would succeed.
2. -o yaml
Outputs the resource definition in YAML format to the terminal.

When used together, these options allow us to generate configuration files instantly. We can then redirect the output to a file and edit it as needed:
```kubectl run nginx --image=nginx --dry-run=client -o yaml > nginx-pod.yaml```

# POD

## Create an NGINX Pod:
```kubectl run nginx --image=nginx```

## Generate the Pod YAML (without creating it):
```kubectl run nginx --image=nginx --dry-run=client -o yaml```

## Create a Pod named redis using the redis:alpine image and assigns it the label tier=db:
```kubectl run redis --image=redis:alpine --labels="tier=db"```

## Create a Pod named httpd running the httpd:alpine image and exposes port 80. It also automatically creates a ClusterIP Service to make the Pod accessible within the cluster.
```kubectl run httpd --image=httpd:alpine --port=80 --expose=true```


# Deployment

## Create a Deployment:
```kubectl create deployment nginx --image=nginx```

## Generate the Deployment YAML (without creating it):
```kubectl create deployment nginx --image=nginx --dry-run=client -o yaml```

## Create a Deployment with 4 replicas:
```kubectl create deployment nginx --image=nginx --replicas=4```

## Scale an existing Deployment:
```kubectl scale deployment nginx --replicas=4```

# Recommended Workflow
## A more flexible approach is to generate the YAML first, modify it, and then apply it:
```kubectl create deployment nginx --image=nginx --dry-run=client -o yaml > nginx-deployment.yaml```

Edit the file (e.g., update replicas or add configurations), then apply it:
```kubectl apply -f nginx-deployment.yaml```
This workflow combines speed with control, allowing you to avoid writing YAML from scratch while still customizing your resources as needed.



# Service
## Create a ClusterIP Service
To create a Service named redis-service of type ClusterIP that exposes the redis pod on port 6379, you can use:
```kubectl expose pod redis --port=6379 --name=redis-service --dry-run=client -o yaml```
Key advantage:
Automatically uses the pod’s existing labels as selectors.

## Alternative Approach
```kubectl create service clusterip redis --tcp=6379:6379 --dry-run=client -o yaml```

Important notes:
- This command does not use the pod’s actual labels.
- It assumes a default selector: app=redis.
- You cannot specify custom selectors directly via the command.

Because of this limitation, you’ll typically need to:
1. Generate the YAML file
2. Manually update the selector field
3. Apply the configuration


## Create a NodePort Service
To create a Service named nginx-service of type NodePort, exposing the nginx pod’s port 80:
```kubectl expose pod nginx --port=80 --name=nginx-service --type=NodePort --dry-run=client -o yaml```
Key advantage:
- Automatically uses the pod’s labels as selectors.
Limitation:
- You cannot specify the nodePort using this command.

## Alternative Approach
```kubectl create service nodeport nginx --tcp=80:80 --node-port=30080 --dry-run=client -o yaml```

Important notes:
- Allows you to specify the nodePort (30080).
- However, it does not use the pod’s actual labels as selectors.


# Recommendation
Both approaches have trade-offs:
```kubectl expose```
✔ Correct selectors automatically
❌ Cannot set 'nodePort'
``` kubectl create service```
✔ Can set 'nodePort'
❌ Uses default (often incorrect) selectors

## Best practice:
Use 'kubectl expose' to generate the YAML file, since it correctly captures the pod’s labels.
If you need to define a specific 'nodePort', simply edit the generated YAML and add it manually before applying:
```kubectl apply -f service.yaml```

This approach gives you both accuracy and flexibility without unnecessary debugging.

# Formatting Output
By default, all kubectl commands return output in a human-readable plain-text format. While this format is useful for quick inspection, Kubernetes also allows you to display the same information in other formats using the -o (output) flag.

## Syntax
```kubectl [command] [TYPE] [NAME] -o <output_format>```
### Common Output Formats
1. ```-o json```
Displays the resource as a JSON-formatted API object. Useful for scripting and integrations.
2. ```-o yaml```
Outputs the resource definition in YAML format. Ideal for editing and reapplying configurations.
3. ```-o wide```
Shows the standard output with additional details (such as node information, IPs, etc.).
4. ```-o name```
Prints only the resource name, which is helpful when chaining commands or writing scripts.


# Kubectl Explain Command:
### List all resource types (Pods, Services, Deployments, etc.) that the cluster API supports, including whether they are namespaced.
```kubectl api-resources```
### Show documentation for the Pod resource, describing its purpose and top-level fields:
```kubectl explain pods```
### Dive into the Pod’s spec section, explaining how to configure containers, volumes, and other Pod specifications.
```kubectl explain pods.spec```
### Show a full, detailed explanation of all Pod fields and subfields recursively, giving complete documentation for the resource structure.
```kubectl explain pods --recursive```
