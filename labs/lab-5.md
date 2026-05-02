# Q1

Set the context to `cluster1`. Create a pod named `mypod` in the namespace `ckad-prep` using the image `redis:7-alpine`. The pod must have the label `env=exam`. Once running, extract the pod's IP address and write it to the file `/tmp/mypod-ip.txt`.

# Solution

```bash
k config use-context cluster1

k run mypod -n ckad-prep --image=redis:7-alpine --labels="env=exam"
```

```bash
k get pod mypod -n ckad-prep -o jsonpath='{.status.podIP}' > /tmp/mypod-ip.txt
```

---

# Q2

A pod named `broken-config` in namespace `ckad-system` is not starting. Without deleting the pod, identify what is wrong. Fix the issue by editing the pod definition, export the fixed YAML to `/tmp/fixed-pod.yaml`, delete the broken pod, and re-create it from the fixed file.

## Key Insight

Pods are **immutable** for many fields (image, env, volumes, etc.).

You **CANNOT fully fix it in-place** → must recreate.

# Solution

```bash
k describe pod broken-config -n ckad-system
```

Look for:

* `ImagePullBackOff`
* `CrashLoopBackOff`
* Missing ConfigMap/Secret

```bash
k get pod broken-config -n ckad-system -o yaml > /tmp/fixed-pod.yaml
vi /tmp/fixed-pod.yaml
```

```bash
k delete pod broken-config -n ckad-system
k apply -f /tmp/fixed-pod.yaml
```

---

# Q3

Create a Deployment named `webapp` in namespace `ckad-deploy` using image `nginx:1.23` with `3` replicas. Update the image to `nginx:1.25` and record the change. Then simulate a bad update by setting the image to `nginx:doesnotexist`. Verify the rollout is stuck, then roll back to the last working revision and confirm all pods are healthy.

# Solution

```bash
k create deployment webapp --image=nginx:1.23 --replicas=3 -n ckad-deploy
```

### Update with change-cause

```bash
k set image deployment/webapp nginx=nginx:1.25 -n ckad-deploy 
```

### Bad update

```bash
k set image deployment/webapp nginx=nginx:doesnotexist -n ckad-deploy
```

```bash
k rollout status deployment/webapp -n ckad-deploy
```

You’ll see:

* `ImagePullBackOff`
* rollout stuck

### Rollback

```bash
k rollout undo deployment/webapp -n ckad-deploy
k rollout status deployment/webapp -n ckad-deploy
```

---

# Q4

A Deployment named `api-server` exists in namespace `ckad-deploy`. Without deleting it, update the deployment so that: it uses a `Recreate` update strategy, the number of replicas is `4`, and the container has environment variable `VERSION=v2` added. Apply all changes in a single kubectl apply command using a patched YAML file.

## Key Concepts

* `Recreate` → **downtime allowed**, all pods killed first
* YAML editing is safest for multi-field changes

# Solution

```bash
k get deployment api-server -n ckad-deploy -o yaml > /tmp/api-server.yaml
vi /tmp/api-server.yaml
```

Add:

```yaml
spec:
  replicas: 4
  strategy:
    type: Recreate
  template:
    spec:
      containers:
      - name: <container-name>
        env:
        - name: VERSION
          value: v2
```

```bash
k apply -f /tmp/api-server.yaml
k rollout status deployment/api-server -n ckad-deploy
```

---

# Q5

Create a ConfigMap named `nginx-conf` in namespace `ckad-design` from the literal value `server_name=myapp.local`. Create a Pod named `config-pod` using image `nginx:1.25` that mounts this ConfigMap as a volume at `/etc/nginx/conf.d`. Additionally inject the same key `server_name` as an environment variable named `SERVER_NAME` using `valueFrom.configMapKeyRef`.

# Solution

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: config-pod
  namespace: ckad-design
spec:
  containers:
  - name: nginx-container
    image: nginx:1.25
    volumeMounts:
    - name: config-volume
      mountPath: /etc/nginx/conf.d

    env:
    - name: SERVER_NAME
      valueFrom:
        configMapKeyRef:
          name: nginx-conf
          key: server_name

  volumes:
  - name: config-volume
    configMap:
      name: nginx-conf
```


---

# Q6

Create a Pod named `logger` in namespace `ckad-design` with two containers. Container `writer` uses image `busybox:1.35` and runs: `sh -c 'while true; do date >> /logs/app.log; sleep 2; done'`. Container `reader` uses image `busybox:1.35` and runs: `sh -c 'tail -f /logs/app.log'`. Both share an `emptyDir` volume `log-share` at `/logs`. Verify the reader is printing timestamps by checking its logs.


# Solution

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: logger
  namespace: ckad-design
spec:
  containers:
  - name: writer
    image: busybox:1.35
    command: ["sh", "-c", "while true; do date >> /logs/app.log; sleep 2; done"]
    volumeMounts:
    - name: log-volume
      mountPath: /logs

  - name: reader
    image: busybox:1.35
    command: ["sh", "-c", "tail -f /logs/app.log"]
    volumeMounts:
    - name: log-volume
      mountPath: /logs

  volumes:
  - name: log-volume
    emptyDir: {}
```

### Verify:

```bash
k logs logger -c reader -n ckad-design
```

---

# Q7

Create a Secret named `db-creds` in namespace `ckad-config` with keys `username=admin` and `password=S3cur3Pass`. Create a Pod named `db-pod` using image `postgres:15-alpine` that: injects `username` as env var `POSTGRES_USER`, injects `password` as env var `POSTGRES_PASSWORD`, and mounts the full secret as a volume at `/etc/db-creds`. Confirm both env vars are set inside the running container.

# Solution
```bash
kubectl create secret generic db-creds -n ckad-config \
  --from-literal=username=admin \
  --from-literal=password=S3cur3Pass
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: db-pod
  namespace: ckad-config
spec:
  volumes:
  - name: creds-vol
    secret:
      secretName: db-creds
  containers:
  - name: postgres
    image: postgres:15-alpine
    env:
    - name: POSTGRES_USER
      valueFrom:
        secretKeyRef:
          name: db-creds
          key: username
    - name: POSTGRES_PASSWORD
      valueFrom:
        secretKeyRef:
          name: db-creds
          key: password
    volumeMounts:
    - name: creds-vol
      mountPath: /etc/db-creds
```

```bash
kubectl exec db-pod -n ckad-config -- env | grep POSTGRES
```

---

# Q8

Create a namespace `ckad-quota` with a ResourceQuota named `ns-quota` that limits: `pods: 5`, `requests.cpu: 1`, `requests.memory: 1Gi`, `limits.cpu: 2`, `limits.memory: 2Gi`. Then create a Deployment named `quota-app` (image `nginx:1.25`, `3` replicas) in that namespace with CPU request `100m`/limit `300m` and memory request `64Mi`/limit `256Mi` per pod. Verify the quota usage.

# Solution
```bash
k create quota ns-quota -n ckad-quota \
--hard=pods=5,requests.cpu=1,requests.memory=1Gi,limits.cpu=2,limits.memory=2Gi
```

OR  

```bash
kubectl create namespace ckad-quota
```

```yaml 
apiVersion: v1
kind: ResourceQuota
metadata:
  name: ns-quota
  namespace: ckad-quota
spec:
  hard:
    pods: "5"
    requests.cpu: "1"
    requests.memory: "1Gi"
    limits.cpu: "2"
    limits.memory: "2Gi"

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: quota-app
  namespace: ckad-quota
spec:
  replicas: 3
  selector:
    matchLabels:
      app: quota-app
  template:
    metadata:
      labels:
        app: quota-app
    spec:
      containers:
      - name: nginx
        image: nginx:1.25
        resources:
          requests:
            cpu: "100m"
            memory: "64Mi"
          limits:
            cpu: "300m"
            memory: "256Mi"
```

# Verify quota consumption
```bash
kubectl describe resourcequota ns-quota -n ckad-quota
```

---

# Q9
A pod named `target-pod` in namespace `ckad-debug` does not have a shell. Use `kubectl debug` to attach an ephemeral container using image `busybox:1.35` to inspect the pod's filesystem. List the contents of `/etc` inside the target pod's filesystem. Then write the process list from inside the pod to `/tmp/target-procs.txt`.

# Solution

### Attach ephemeral debug container
```bash
kubectl debug -it target-pod -n ckad-debug \
  --image=busybox:1.35 \
  --target=<main-container-name> \
  -- sh
```

### Inside the ephemeral container:
```bash
ls /etc
ps aux
```

### From outside, write ps output to file (run ps via exec after debug session):
```bash
kubectl exec target-pod -n ckad-debug -- ps aux > /tmp/target-procs.txt 2>/dev/null || \
kubectl debug target-pod -n ckad-debug --image=busybox:1.35 -- ps aux > /tmp/target-procs.txt
```

---

# Q10

Find the pod consuming the most CPU across all namespaces and write its name and namespace to `/tmp/top-cpu-pod.txt` in the format `namespace/podname`. Then find the node with the highest memory usage and write its name to `/tmp/top-mem-node.txt`. If metrics-server is unavailable, use `kubectl describe nodes` to estimate allocatable memory instead.

# Solution

### Top CPU pod across all namespaces
```bash
kubectl top pods -A --sort-by=cpu | head -2
```

### Write top result (first data row after header)
```bash
kubectl top pods -A --sort-by=cpu --no-headers | head -1 | \
  awk '{print $1"/"$2}' > /tmp/top-cpu-pod.txt

cat /tmp/top-cpu-pod.txt
```

### Top memory node
```bash
kubectl top nodes --sort-by=memory --no-headers | head -1 | \
  awk '{print $1}' > /tmp/top-mem-node.txt

cat /tmp/top-mem-node.txt
```

### Fallback if metrics-server unavailable:
```bash
kubectl describe nodes | grep -A5 "Allocated resources"
```

---

# Q11

A Deployment named `front-end` (label `app=front-end, containerPort 80`) exists in namespace `ckad-svc`. Create a Service named `front-svc` of type NodePort that maps port `80` to nodePort `30080`. Then create a second Service named `front-internal` of type `ClusterIP` on port `8080` targeting port `80`. Verify both services list endpoints.

# Solution
```yaml
# NodePort service
apiVersion: v1
kind: Service
metadata:
  name: front-svc
  namespace: ckad-svc
spec:
  type: NodePort
  selector:
    app: front-end
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30080

---
# ClusterIP service
apiVersion: v1
kind: Service
metadata:
  name: front-internal
  namespace: ckad-svc
spec:
  type: ClusterIP
  selector:
    app: front-end
  ports:
  - port: 8080
    targetPort: 80
```

```bash
kubectl apply -f services.yaml
```

# Verify endpoints
```bash
kubectl get endpoints front-svc front-internal -n ckad-svc
```

---

# Q12

In namespace `ckad-net`, pods with label `app=payments` must only accept ingress from: (a) pods labelled `app=checkout` within the same namespace, and (b) pods in any namespace that carry the label `tier=monitoring`. All other ingress must be blocked. Egress must remain unrestricted. Create the NetworkPolicy named `payments-policy`.

# Solution
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: payments-policy
  namespace: ckad-net
spec:
  podSelector:
    matchLabels:
      app: payments
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:          # (a) same namespace, app=checkout
        matchLabels:
          app: checkout
  - from:
    - namespaceSelector: {} # any namespace...
      podSelector:          # ...but only pods with tier=monitoring
        matchLabels:
          tier: monitoring
```

### Note: two separate 'from' list items = OR logic
### Combined namespaceSelector+podSelector in same item = AND logic
```bash
kubectl apply -f payments-policy.yaml
kubectl describe netpol payments-policy -n ckad-net
```

---

# Q13

Node `node02` needs maintenance. Safely drain it so all workloads are rescheduled: ignore DaemonSet-managed pods, forcefully evict pods with no controller (use `--force` and `--delete-emptydir-data`). After draining, confirm the node is cordoned. Write the list of pods that were evicted during drain to `/tmp/drained-pods.txt`. Finally, uncordon the node.

# Solution
### List pods on the node before draining
```bash
kubectl get pods -A --field-selector=spec.nodeName=node02 > /tmp/drained-pods.txt
```

### Drain the node
```bash
kubectl drain node02 \
  --ignore-daemonsets \
  --force \
  --delete-emptydir-data
```

### Confirm node is cordoned (SchedulingDisabled)
```bash
kubectl get node node02
```

### Do maintenance work here...

### Uncordon when done
```bash
kubectl uncordon node02

kubectl get node node02
```


---

# Q14

Create a ServiceAccount named `pipeline-sa` in namespace `ckad-rbac`. Create a ClusterRole named `pod-reader` that allows `get`, `watch`, and `list` on `pods` and `pods/log` in the core API group. Bind this ClusterRole to the ServiceAccount using a ClusterRoleBinding named `pipeline-binding`. Verify the ServiceAccount can list pods in namespace `kube-system` but cannot delete them.

# Solution
```bash
kubectl create namespace ckad-rbac
kubectl create serviceaccount pipeline-sa -n ckad-rbac
```

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods","pods/log"]
  verbs: ["get","watch","list"]

---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: pipeline-binding
subjects:
- kind: ServiceAccount
  name: pipeline-sa
  namespace: ckad-rbac
roleRef:
  kind: ClusterRole
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

```bash
kubectl apply -f rbac.yaml

### Verify — should return 'yes'
kubectl auth can-i list pods -n kube-system \
  --as=system:serviceaccount:ckad-rbac:pipeline-sa

### Should return 'no'
kubectl auth can-i delete pods -n kube-system \
  --as=system:serviceaccount:ckad-rbac:pipeline-sa
```

---

# Q15
Create a PersistentVolumeClaim named app-pvc in namespace ckad-storage requesting 500Mi of storage with access mode ReadWriteOnce using the default StorageClass. Create a Deployment named storage-app (image: nginx:1.25, 1 replica) that mounts this PVC at /usr/share/nginx/html. Exec into the running pod and create a file /usr/share/nginx/html/index.html with content Hello CKAD. Delete the pod (not the deployment) and verify the file persists when the replacement pod starts.

# Solution

### Check available storage classes
```bash
kubectl get storageclass
```

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: app-pvc
  namespace: ckad-storage
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 500Mi
  # storageClassName: standard  # omit to use cluster default

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: storage-app
  namespace: ckad-storage
spec:
  replicas: 1
  selector:
    matchLabels:
      app: storage-app
  template:
    metadata:
      labels:
        app: storage-app
    spec:
      volumes:
      - name: web-storage
        persistentVolumeClaim:
          claimName: app-pvc
      containers:
      - name: nginx
        image: nginx:1.25
        volumeMounts:
        - name: web-storage
          mountPath: /usr/share/nginx/html
```

```bash
kubectl apply -f storage.yaml

### Write file into running pod
kubectl exec -n ckad-storage deploy/storage-app -- \
  sh -c 'echo "Hello CKAD" > /usr/share/nginx/html/index.html'

### Delete the pod (deployment will recreate it)
kubectl delete pod -n ckad-storage -l app=storage-app

### Wait for new pod, then verify file persists
kubectl exec -n ckad-storage deploy/storage-app -- \
  cat /usr/share/nginx/html/index.html
```