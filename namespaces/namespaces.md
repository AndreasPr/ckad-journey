The specific YAML defines a Namespace named 'dev' in Kubernetes.
A Namespace is used to logically separate and organize resources within a cluster.
Resources created in this namespace (e.g., Pods, Deployments) will be isolated from those in other namespaces.
```
apiVersion: v1 
kind: Namespace 
metadata:
    name: dev
```


## Resource Quota (ex. compute-quota.yaml)
#### The YAML defines a ResourceQuota named compute-quota in the dev namespace, limiting total resource usage (e.g., max 10 Pods, 4 CPU requests, 10 CPU limits, 5Gi memory requests, 10Gi memory limits):
```
apiVersion: v1
kind: ResourceQuota
metadata:
    name: compute-quota
    namespace: dev
spec:
    hard:
        pods: "10"
        requests.cpu: "4"
        requests.memory: 5Gi
        limits.cpu: "10"
        limits.memory: 10Gi
```
### Apply this quota to the cluster, enforcing these limits on all resources created in the dev namespace:
```kubectl create -f compute-quota.yaml```

## Example: ```db-service.dev.svc.cluster.local``` is the internal DNS name used in Kubernetes to reach a Service to a different namespace than your current one:
- db-service -> the Service name
- dev -> the Namespace
- svc -> indicates it’s a Service
- cluster.local -> the cluster’s internal DNS domain
Together, it allows Pods inside the cluster to connect to the db-service in the dev namespace using stable DNS instead of IP addresses.

## Commands
### Create a new namespace called dev:
```kubectl create -f namespace-dev.yaml ```
or
```kubectl create namespace dev```

### List pods only in the dev namespace:
```kubectl get pods --namespace=dev```
or 
```kubectl get pods -n=dev```

### Set 'dev' as the default namespace for future commands:
```kubectl config set-context $(kubectl config current-context) --namespace=dev```

### List Pods across all namespaces in the cluster:
```kubectl get pods --all-namespaces```
or
```kubectl get pods -A```

### List all namespaces in the cluster:
```kubectl get namespaces```
or
```kubectl get ns```