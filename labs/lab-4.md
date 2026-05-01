## Q1
Create a Secret named `tls-secret` in namespace `secure` of type kubernetes.io/tls using existing files `/certs/tls.crt` and `/certs/tls.key`. Mount this secret into a Pod named `tls-pod` (image: `nginx:1.25`) at `/etc/nginx/certs`. Set the mount as read-only. Confirm the files are present inside the container.

### Key Concepts
- TLS secrets are a special type with keys tls.crt and tls.key
- Commonly used by Ingress controllers and HTTPS workloads
- Mounting secrets exposes them as files inside containers

### Solution
```bash
kubectl create namespace secure

kubectl create secret tls tls-secret \
  -n secure \
  --cert=/certs/tls.crt \
  --key=/certs/tls.key
````

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: tls-pod
  namespace: secure
spec:
  volumes:
  - name: certs
    secret:
      secretName: tls-secret

  containers:
  - name: nginx
    image: nginx:1.25
    volumeMounts:
    - name: certs
      mountPath: /etc/nginx/certs
      readOnly: true
```

Verify files inside container:

```bash
kubectl exec -it tls-pod -n secure -- ls /etc/nginx/certs
```

---

## Q2

Create a Pod named `projected-pod` using image `busybox:1.35` that runs `sleep 3600`. Use a projected volume named `combined-vol` mounted at `/config` that combines: a ConfigMap named `app-cm` (key: `APP_ENV=staging`) and a Secret named `app-sec` (key: `PASSWORD=hunter2`). Both must appear as files at `/config`.

### Key Concepts

* Projected volumes merge multiple sources into a single mount
* Useful when apps expect all config in one directory

### Solution

```bash
kubectl create configmap app-cm --from-literal=APP_ENV=staging

kubectl create secret generic app-sec --from-literal=PASSWORD=hunter2
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: projected-pod
spec:
  containers:
  - name: busybox
    image: busybox:1.35
    command: ["sleep", "3600"]
    volumeMounts:
    - name: combined-vol
      mountPath: /config

  volumes:
  - name: combined-vol
    projected:
      sources:
      - configMap:
          name: app-cm
      - secret:
          name: app-sec
```

Verify:

```bash
kubectl exec projected-pod -- ls /config
```

---

## Q3

Create a Pod named `secure-ctx` using image `nginx:1.25` with the following security requirements: the pod must run as user `1000` and group `3000`; the container must run as non-root; drop capability ALL and add only `NET_BIND_SERVICE`; set the root filesystem to read-only with an `emptyDir` volume at `/tmp` to allow writes.

### Key Concepts

* runAsUser, runAsGroup enforce identity
* runAsNonRoot prevents root execution
* readOnlyRootFilesystem improves security
* capabilities restrict Linux privileges

### Solution

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secure-ctx
spec:
  securityContext:
    runAsUser: 1000
    runAsGroup: 3000
    fsGroup: 3000

  volumes:
  - name: tmp-vol
    emptyDir: {}

  containers:
  - name: nginx
    image: nginx:1.25
    volumeMounts:
    - name: tmp-vol
      mountPath: /tmp
    securityContext:
      runAsNonRoot: true
      readOnlyRootFilesystem: true
      capabilities:
        drop: ["ALL"]
        add: ["NET_BIND_SERVICE"]
```

---

## Q4

Create a LimitRange named `cpu-mem-limits` in namespace `restricted` that enforces: default CPU request `100m`, default CPU limit `500m`, default memory request `64Mi`, default memory limit `256Mi`. Then create a Pod named `limited-pod` (image: `nginx:1.25`) in that namespace without specifying any resources. Verify the LimitRange defaults were applied.

### Key Concepts

* LimitRange enforces default resource requests/limits
* Applied automatically if Pod doesn't define them

### Solution

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: cpu-mem-limits
  namespace: restricted
spec:
  limits:
  - type: Container
    default:
      cpu: 500m
      memory: 256Mi
    defaultRequest:
      cpu: 100m
      memory: 64Mi
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: limited-pod
  namespace: restricted
spec:
  containers:
  - name: nginx
    image: nginx:1.25
```

Verify limits were injected:

```bash
kubectl describe pod limited-pod -n restricted
```

---

## Q5

A Deployment named `compute` (image: `nginx:1.25`, `2` replicas) exists in default. Create a HorizontalPodAutoscaler named `compute-hpa` that scales the deployment between `2` and `8` replicas, targeting `60%` average CPU utilization. The deployment containers must have a CPU request set to `200m` (update it if not set). Verify the HPA is created.

### Key Concepts

* HPA scales based on metrics (CPU here)
* Requires resource requests to be set
* Uses metrics-server in cluster

### Solution

First ensure CPU request is set:

```bash
kubectl set resources deployment compute --requests=cpu=200m
```

Create HPA:

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: compute-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: compute
  minReplicas: 2
  maxReplicas: 8
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 60
```

Verify:

```bash
kubectl get hpa compute-hpa
```

---

## Q6

Implement a canary deployment pattern. A Deployment `app-stable` (image: `myapp:1.0`, `4` replicas, label `app=myapp`) and a Service `app-svc` selecting `app=myapp` already exist. Create a canary Deployment named `app-canary` (image: `myapp:2.0`, `1` replica) that also carries label `app=myapp` so the Service routes ~20% of traffic to it. Add a distinguishing label `track=canary` to the canary pods.

### Key Concepts

* Traffic split is based on replica ratio
* Service routes based on labels only
* Canary adds a small number of replicas

### Solution

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-canary
spec:
  replicas: 1
  selector:
    matchLabels:
      app: myapp
      track: canary
  template:
    metadata:
      labels:
        app: myapp
        track: canary
    spec:
      containers:
      - name: myapp
        image: myapp:2.0
```

Verify both deployments feed the service:

```bash
kubectl get endpoints app-svc
```
* Should show 5 IPs total (4 stable + 1 canary)

---

## Q7

Create a Job named `batch-process` that runs `6` completions with a parallelism of `2`. Use image `busybox:1.35` with command `sh -c 'echo processing item; sleep 5; exit 0'`. Set `backoffLimit: 2` and `activeDeadlineSeconds: 120`. Monitor the job until all completions finish.

### Key Concepts

* completions = total runs
* parallelism = concurrent runs
* backoffLimit controls retries
* activeDeadlineSeconds enforces timeout

### Solution

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: batch-process
spec:
  completions: 6
  parallelism: 2
  backoffLimit: 2
  activeDeadlineSeconds: 120
  template:
    spec:
      containers:
      - name: busybox
        image: busybox:1.35
        command: ["sh", "-c", "echo processing; sleep 5"]
      restartPolicy: Never
```

Monitor until complete:

```bash
kubectl get jobs -w
```

---

## Q8

Create a Deployment named `probe-app` (image: `nginx:1.25`, `2` replicas) with all three probe types configured on the container: a startupProbe (HTTP `GET` `/` port `80`, `failureThreshold 10`, period `5s`), a livenessProbe (HTTP `GET` `/` port `80`, initialDelay `15s`, period `20s`), and a readinessProbe (HTTP `GET` `/healthz` port `80`, initialDelay `5s`, period `10s`).

### Key Concepts

* startupProbe: initial boot protection
* livenessProbe: restart unhealthy containers
* readinessProbe: control traffic routing

### Solution

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: probe-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: probe-app
  template:
    metadata:
      labels:
        app: probe-app
    spec:
      containers:
      - name: nginx
        image: nginx:1.25
        ports:
        - containerPort: 80

        startupProbe:
          httpGet:
            path: /
            port: 80
          failureThreshold: 10
          periodSeconds: 5

        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 15
          periodSeconds: 20

        readinessProbe:
          httpGet:
            path: /healthz
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 10
```

---

## Q9

A Service `web-svc` (port `80`) and Service `api-svc` (port `8080`) exist in namespace `ingress-test`. Create an Ingress named `app-ingress` using ingress class `nginx` that routes: traffic to host `app.example.com` with path `/` to `web-svc:80`, and path `/api` to `api-svc:8080`. Use `pathType: Prefix`.

### Key Concepts

* Ingress routes HTTP traffic based on host/path
* Requires ingress controller installed

### Solution

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  namespace: ingress-test
spec:
  ingressClassName: nginx
  rules:
  - host: app.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-svc
            port:
              number: 80
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: api-svc
            port:
              number: 8080
```

---

## Q10

In namespace `net-test`, implement a default-deny-all ingress policy for all pods. Then create a second NetworkPolicy named `allow-prometheus` that allows ingress on port `9090` to pods with label `app=metrics` only from pods with label `role=prometheus` in the `monitoring` namespace. Both policies must coexist.

### Key Concepts

* NetworkPolicies are additive
* Default deny = no ingress unless allowed

### Solution

Default deny-all ingress for all pods in namespace:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny
  namespace: net-test
spec:
  podSelector: {}
  policyTypes:
  - Ingress
```

Allow prometheus from monitoring namespace:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-prometheus
  namespace: net-test
spec:
  podSelector:
    matchLabels:
      app: metrics
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: monitoring
      podSelector:
        matchLabels:
          role: prometheus
    ports:
    - port: 9090
```

---

## Q11

A Deployment named `memory-hog` in namespace `default` has pods that are being OOMKilled. Investigate: retrieve the last termination reason and exit code for any pod in this deployment. Retrieve logs from the previous container instance. Write a one-line summary of the cause to `/tmp/oom-report.txt`. Then patch the deployment to increase the memory limit to `512Mi`.

### Key Concepts

* Exit code 137 = OOMKilled
* --previous shows crash logs
* Fix by increasing memory limits

### Solution

Find a pod in the deployment:

```bash
kubectl get pods -l app=memory-hog
```

Get termination reason:

```bash
kubectl describe pod <pod> | grep -A5 "Last State"
```

Get previous logs:

```bash
kubectl logs <pod> --previous > /tmp/prev.log
```

Write summary:

```bash
echo "OOMKilled due to memory limit exceeded" > /tmp/oom-report.txt
```

Patch the deployment memory limit:

```bash
kubectl set resources deployment memory-hog --limits=memory=512Mi
```

Verify rollout:

```bash
kubectl rollout status deployment memory-hog
```

---

## Q12

List all Pods across all namespaces and output a custom table with columns: `NAMESPACE, NAME, STATUS, and NODE`. Sort the output by namespace. Save this to `/tmp/pods-report.txt`. Then write a second command that outputs only the names of pods that are NOT in Running status across all namespaces.

### Key Concepts

* custom-columns formats output
* field-selector filters results

### Solution

Custom columns output sorted by namespace:

```bash
kubectl get pods -A \
  -o custom-columns="NAMESPACE:.metadata.namespace,NAME:.metadata.name,STATUS:.status.phase,NODE:.spec.nodeName" \
  --sort-by=.metadata.namespace \
  > /tmp/pods-report.txt
```

List only non-Running pods:

```bash
kubectl get pods -A --field-selector=status.phase!=Running \
  -o custom-columns="NAMESPACE:.metadata.namespace,NAME:.metadata.name,STATUS:.status.phase"
```

---

## Q13

A Deployment `critical-app` (`5` replicas, label `app=critical`) runs in namespace `default`. Create a PodDisruptionBudget named `critical-pdb` that ensures at least `3` pods are always available during voluntary disruptions. Then update the Deployment's update strategy to RollingUpdate with `maxUnavailable: 1` and `maxSurge: 1` to align with the PDB.

### Key Concepts

* PDB protects availability during disruptions
* Works with voluntary disruptions only

### Solution

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: critical-pdb
spec:
  minAvailable: 3
  selector:
    matchLabels:
      app: critical
```

Patch deployment rolling update strategy:

```bash
kubectl patch deployment critical-app -p '{
  "spec": {
    "strategy": {
      "type": "RollingUpdate",
      "rollingUpdate": {
        "maxUnavailable": 1,
        "maxSurge": 1
      }
    }
  }
}'
```

Verify PDB:

```bash
kubectl get pdb critical-pdb
kubectl describe pdb critical-pdb
```

---

## Q14

Create a ServiceAccount named `ci-runner` in namespace `ci`. Create a Role named `deploy-role` in namespace `ci` that allows `get, list, update, and patch` on deployments and replicasets in the `apps` API group. Bind this Role to the ServiceAccount using a RoleBinding named `ci-binding`. Verify the permissions using `kubectl auth can-i`.

### Key Concepts

* Role = permissions
* RoleBinding = attach role to subject
* ServiceAccount = identity

### Solution

```bash
kubectl create namespace ci
kubectl create serviceaccount ci-runner -n ci
```

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: deploy-role
  namespace: ci
rules:
- apiGroups: ["apps"]
  resources: ["deployments", "replicasets"]
  verbs: ["get", "list", "update", "patch"]
```

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: ci-binding
  namespace: ci
subjects:
- kind: ServiceAccount
  name: ci-runner
roleRef:
  kind: Role
  name: deploy-role
  apiGroup: rbac.authorization.k8s.io
```

Verify permissions:

```bash
kubectl auth can-i update deployments \
  -n ci \
  --as=system:serviceaccount:ci:ci-runner
```

```bash
kubectl auth can-i delete deployments -n ci \
  --as=system:serviceaccount:ci:ci-runner  # should return 'no'
```

---

## Q15

Create a headless Service named `db-headless` in namespace `default` with selector `app=db` and port `5432`. Then create a StatefulSet named `postgres` with `3` replicas using image `postgres:15-alpine`, the env var `POSTGRES_PASSWORD=secret`, and a volumeClaimTemplate that provisions a `1Gi` PVC named data using `ReadWriteOnce` mounted at `/var/lib/postgresql/data`. Verify stable pod DNS is reachable.

### Key Concepts

* StatefulSets provide stable identities
* Headless Service enables DNS per pod
* volumeClaimTemplates create per-pod storage

### Solution

Headless Service:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: db-headless
spec:
  clusterIP: None
  selector:
    app: db
  ports:
  - port: 5432
```

StatefulSet:

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres
spec:
  serviceName: db-headless
  replicas: 3
  selector:
    matchLabels:
      app: db
  template:
    metadata:
      labels:
        app: db
    spec:
      containers:
      - name: postgres
        image: postgres:15-alpine
        env:
        - name: POSTGRES_PASSWORD
          value: secret
        volumeMounts:
        - name: data
          mountPath: /var/lib/postgresql/data
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 1Gi
```

Verify stable DNS from another pod:

```bash
kubectl run tmp --image=busybox:1.35 --rm -it --restart=Never -- \
  nslookup postgres-0.db-headless.default.svc.cluster.local
```
