## Q1
Create a Pod named `web-pod` in the `default` namespace using image `nginx:1.25`. Set the label `tier=frontend` and the environment variable `APP_MODE=production`

### Key Concepts
- Labels are used for grouping and selecting Kubernetes resources
- Environment variables configure runtime behavior inside containers
- kubectl run can quickly create simple Pods without YAML

### Solution
```bash
kubectl run web-pod \
  --image=nginx:1.25 \
  --labels="tier=frontend" \
  --env="APP_MODE=production" \
  --restart=Never
````

### Or via manifest:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-pod
  namespace: default
  labels:
    tier: frontend
spec:
  containers:
  - name: web-pod
    image: nginx:1.25
    env:
    - name: APP_MODE
      value: production
```

---

## Q2

Create a Namespace named `team-alpha`. Then create a Pod named `alpha-pod` in that namespace using image `busybox:1.35` that runs the command `sleep 3600`. Verify the pod is running.

### Key Concepts

* Namespaces isolate resources logically
* Always specify namespace explicitly in exam tasks
* Verification is part of the workflow

### Solution

```bash
kubectl create namespace team-alpha

kubectl run alpha-pod \
  -n team-alpha \
  --image=busybox:1.35 \
  --restart=Never \
  --command -- sleep 3600

kubectl get pod alpha-pod -n team-alpha
```

---

## Q3

Create a Deployment named `backend` with `4` replicas using image `node:18-alpine`. The container must expose port `3000` and have the label `app=backend`. After creation, scale it down to `2` replicas.

### Key Concepts

* Deployments manage replica sets and scaling
* Labels are critical for service selection
* Scaling can be done declaratively or imperatively

### Solution

```bash
kubectl create deployment backend \
  --image=node:18-alpine \
  --replicas=4 \
  --port=3000

kubectl label deployment backend app=backend

kubectl scale deployment backend --replicas=2
```

---

## Q4

A Deployment named `api` exists in namespace `staging`. Perform a rolling update to image `myapi:3.0`. Annotate the rollout with the change-cause update to `v3.0`. Then verify the rollout completed successfully.

### Key Concepts

* Rolling updates ensure zero downtime deployments
* Annotations help track change history
* Rollout status shows progress and success

### Solution

```bash
kubectl set image deployment/api api=myapi:3.0 -n staging

kubectl annotate deployment/api \
  kubernetes.io/change-cause="update to v3.0" \
  -n staging

kubectl rollout status deployment/api -n staging
kubectl rollout history deployment/api -n staging
```

---

## Q5

The Deployment `frontend` in namespace `default` was just updated and is broken. Roll it back to the previous revision. Confirm the rollback is complete.

### Key Concepts

* Rollbacks revert to a stable state
* Deployment revisions are automatically tracked

### Solution

```bash
kubectl rollout undo deployment/frontend

kubectl rollout status deployment/frontend
```

---

## Q6

Create a Pod named `init-pod` using image `nginx:1.25` as the main container. Add an init container that writes a file to a shared volume.

### Key Concepts

* Init containers run before main containers
* Used for setup tasks like configuration or data preparation
* Must complete successfully before main container starts

### Solution

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: init-pod
spec:
  volumes:
  - name: work-vol
    emptyDir: {}

  initContainers:
  - name: init-setup
    image: busybox:1.35
    command: ["sh", "-c", "echo init complete > /work/status.txt"]
    volumeMounts:
    - name: work-vol
      mountPath: /work

  containers:
  - name: nginx
    image: nginx:1.25
    volumeMounts:
    - name: work-vol
      mountPath: /work
```

---

## Q7

Create a Pod named `sidecar-pod` with a main container app (image: `nginx:1.25`) and a sidecar container `log-agent` (image: `busybox:1.35`) that runs: `sh -c 'while true; do wc -l /logs/access.log 2>/dev/null; sleep 15; done'`. Share a volume named `log-vol` between them mounted at `/logs`.

### Key Concepts

* Sidecar pattern = helper container alongside main app
* Shared volumes enable communication between containers
* Sidecars are regular containers (not initContainers)

### Solution

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sidecar-pod
  namespace: default
spec:
  volumes:
  - name: log-vol
    emptyDir: {}

  containers:
  - name: app
    image: nginx:1.25
    volumeMounts:
    - name: log-vol
      mountPath: /logs

  - name: log-agent
    image: busybox:1.35
    command:
    - sh
    - -c
    - while true; do wc -l /logs/access.log 2>/dev/null; sleep 15; done
    volumeMounts:
    - name: log-vol
      mountPath: /logs
```

---

## Q8

Create a ConfigMap named `db-config` in namespace `prod` with keys `DB_HOST=mysql.prod.svc` and `DB_PORT=3306`. Then create a Pod named `app-pod` using image `busybox:1.35` that loads all ConfigMap keys as environment variables and runs `sh -c 'echo $DB_HOST:$DB_PORT; sleep 3600'`.

### Key Concepts

* ConfigMaps store non-sensitive configuration
* envFrom loads all keys as environment variables
* Useful for decoupling config from code

### Solution

```bash
kubectl create namespace prod

kubectl create configmap db-config -n prod \
  --from-literal=DB_HOST=mysql.prod.svc \
  --from-literal=DB_PORT=3306
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
  namespace: prod
spec:
  containers:
  - name: app
    image: busybox:1.35
    command: ["sh", "-c", "echo $DB_HOST:$DB_PORT; sleep 3600"]
    envFrom:
    - configMapRef:
        name: db-config
```

---

## Q9

Create a Secret named `api-secret` in namespace `secure` with keys `API_KEY=s3cr3tkey` and `API_TOKEN=tok3n99`. Mount this Secret as a volume in a Pod named `secure-pod` (image: `nginx:1.25`) at path `/etc/secrets`. The pod should be `read-only` on that volume.

### Key Concepts

* Secrets store sensitive data
* Mounted as files inside containers
* Can be set as read-only for security

### Solution

```bash
kubectl create namespace secure

kubectl create secret generic api-secret -n secure \
  --from-literal=API_KEY=s3cr3tkey \
  --from-literal=API_TOKEN=tok3n99
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secure-pod
  namespace: secure
spec:
  volumes:
  - name: secret-volume
    secret:
      secretName: api-secret

  containers:
  - name: nginx
    image: nginx:1.25
    volumeMounts:
    - name: secret-volume
      mountPath: /etc/secrets
      readOnly: true
```

---

## Q10

Create a ServiceAccount named `deploy-sa` in the `default` namespace. Create a Pod named `sa-pod` using image `nginx:1.25` that uses this ServiceAccount. Confirm which ServiceAccount is bound to the running pod.

### Key Concepts

* ServiceAccounts provide identity for Pods
* Used for API access control via RBAC

### Solution

```bash
kubectl create serviceaccount deploy-sa
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sa-pod
spec:
  serviceAccountName: deploy-sa
  containers:
  - name: nginx
    image: nginx:1.25
```

Verify:

```bash
kubectl describe pod sa-pod | grep "Service Account"
```

---

## Q11

Create a Pod named `resource-pod` using image `nginx:1.25`. Set resource requests of `cpu: 100m` and `memory: 128Mi`, and limits of `cpu: 500m` and `memory: 256Mi`. Verify the pod is scheduled and display its resource config.

### Key Concepts

* Requests = guaranteed resources
* Limits = maximum allowed usage
* Scheduler uses requests to place Pods

### Solution

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: resource-pod
spec:
  containers:
  - name: nginx
    image: nginx:1.25
    resources:
      requests:
        cpu: "100m"
        memory: "128Mi"
      limits:
        cpu: "500m"
        memory: "256Mi"
```

Verify:

```bash
kubectl describe pod resource-pod
```

---

## Q12

A Deployment named `web` (label `app=web`, port `80`) exists in the default namespace. Create a ClusterIP Service named `web-svc` that exposes port `80` targeting port `80` on pods with label `app=web`. Verify the service has endpoints.

### Key Concepts

* Services provide stable networking
* ClusterIP exposes internally
* Selectors match Pods via labels

### Solution

```bash
kubectl expose deployment web \
  --name=web-svc \
  --port=80 \
  --target-port=80 \
  --type=ClusterIP

kubectl get endpoints web-svc
```

### Or via YAML:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-svc
spec:
  selector:
    app: web
  ports:
  - port: 80
    targetPort: 80
  type: ClusterIP
```

---

## Q13

Pods with label `app=api` exist in namespace `net-test`. Create a NetworkPolicy named `api-ingress` that allows ingress traffic to those pods only from pods with label `role=client` in the same namespace, and only on port `8080`. All other ingress must be denied.

### Key Concepts

* NetworkPolicies control Pod-level networking
* Default deny unless explicitly allowed

### Solution

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: api-ingress
  namespace: net-test
spec:
  podSelector:
    matchLabels:
      app: api
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          role: client
    ports:
    - port: 8080
      protocol: TCP
```

---

## Q14

A Pod named `crash-pod` in namespace `debug` is stuck in CrashLoopBackOff. Retrieve the logs from the previous (crashed) container instance. Save the last `20` lines of those logs to `/tmp/crash-pod.log`. Then describe the pod and identify the exit code.

### Key Concepts

* --previous shows logs from last crash
* describe shows exit codes and failure reasons

### Solution

```bash
kubectl logs crash-pod -n debug --previous --tail=20 > /tmp/crash-pod.log

kubectl describe pod crash-pod -n debug
```

---

## Q15

Create a CronJob named `report-job` in namespace default that runs every `10` minutes using image `busybox:1.35` and executes `date; echo 'report generated'`. Set `concurrencyPolicy: Forbid`, keep `2` successful job histories and `1` failed job history. Manually trigger a Job from it immediately and verify it completes.

### Key Concepts

* CronJobs schedule Jobs
* concurrencyPolicy controls overlapping runs
* History limits control retained jobs

### Solution

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: report-job
spec:
  schedule: "*/10 * * * *"
  concurrencyPolicy: Forbid
  successfulJobsHistoryLimit: 2
  failedJobsHistoryLimit: 1
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: busybox
            image: busybox:1.35
            command:
            - /bin/sh
            - -c
            - date; echo report generated
          restartPolicy: OnFailure
```

Trigger manually:

```bash
kubectl create job report-now --from=cronjob/report-job

kubectl get jobs
kubectl logs job/report-now
```