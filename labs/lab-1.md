# Tasks

## Task 1

Create a PersistentVolume named `log-volume` with the following specifications:

- StorageClassName: `manual` (already created for you — do not create it again)
- AccessModes: `ReadWriteMany` (RWX)
- Capacity: `1Gi`
- hostPath: `/opt/volume/nginx`

Next, create a PersistentVolumeClaim named `log-claim` that:

- Requests at least `200Mi` of storage
- Binds to the `log-volume` created above

Finally, create a pod named `logger` that:

- Uses the image `nginx:alpine`
- Mounts the log-claim at the path `/var/www/nginx` inside the container

Note: This lab environment supports using RWX with hostPath for learning purposes.

Unless otherwise specified, create all resources in the default namespace.

## Solution

Create the persistent volume `log-volume.yaml`:

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: log-volume
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteMany
  storageClassName: manual
  hostPath:
    path: /opt/volume/nginx
```

Create the persistent volume claim `log-claim.yaml`:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: log-claim
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 200Mi
  storageClassName: manual
```

Check the bind status of PV and PVC by running the following command:

```bash
root@controlplane:~$ kubectl get pv,pvc
```

Create the pod `logger.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: logger
  name: logger
spec:
  containers:
  - image: nginx:alpine
    name: logger
    volumeMounts:
    - name: log
      mountPath: /var/www/nginx
  volumes:
  - name: log
    persistentVolumeClaim:
        claimName: log-claim
```


## Task 2
We have already deployed:

- A pod named `secure-pod`
- A service named `secure-service` that targets this pod

Currently, both incoming and outgoing network connections to/from `secure-pod` are failing.

Your task is to troubleshoot and fix the issue so that:

Incoming connections from the pod `webapp-color` to `secure-pod` are successful.

**Requirements**

- Do not delete or recreate existing Kubernetes objects unless the instructions specifically ask you to.
- All resources are located in the default namespace (unless explicitly stated otherwise).
- The fix must be persistent — your changes should remain valid and functional even after testing is repeated.

**Notes**

- Confirm that the `secure-service` is correctly configured and targets the intended pod(s).
- Review any *NetworkPolicies* in the default namespace that could be blocking traffic.
- If there are any conflicting or overlapping NetworkPolicies (such as a `default-deny` policy) that block this connection, modify or remove them so traffic is allowed as required.
- Update or create rules to explicitly allow inbound traffic from the `webapp-color` pod to the `secure-pod` over `TCP` port `80`.


## Solution 

Incoming or outgoing connections are not working because of network policy. In the default namespace, we deployed a `default-deny` network policy which is interrupting the incoming or outgoing connections.

Now, create a network policy called `test-network-policy` to allow the connections, as follows:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: test-network-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      run: secure-pod
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          name: webapp-color
    ports:
    - protocol: TCP
      port: 80
```

then check the connectivity from the `webapp-color` pod to the `secure-pod`:

```bash
root@controlplane:~$ kubectl exec -it webapp-color -- sh
/opt # nc -v -z -w 5 secure-service 80
```


## Task 3
Create a pod named `time-check` in the `dvl1987` namespace. This pod should execute a container called `time-check` using the `busybox` image.

- Create a ConfigMap named `time-config` with the data `TIME_FREQ=10` in the same namespace.
- The `time-check` container must run the command: ```while true; do date; sleep $TIME_FREQ; done```, directing the output to the file located at `/opt/time/time-check.log`.
- Ensure that the path `/opt/time` within the pod mounts a volume that persists for the duration of the pod's lifecycle.


## Solution

Create a namespace called `dvl1987` by using the below command:

```bash 
kubectl create namespace dvl1987
```

Solution manifest file to create a configMap called `time-config` in the given namespace as follows:

```yaml
apiVersion: v1
data:
  TIME_FREQ: "10"
kind: ConfigMap
metadata:
  name: time-config
  namespace: dvl1987
```

Now, create a pod called `time-check` in the same namespace as follows:

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: time-check
  name: time-check
  namespace: dvl1987
spec:
  volumes:
  - name: log-volume
    emptyDir: {}
  containers:
  - image: busybox
    name: time-check
    env:
    - name: TIME_FREQ
      valueFrom:
            configMapKeyRef:
              name: time-config
              key: TIME_FREQ
    volumeMounts:
    - mountPath: /opt/time
      name: log-volume
    command:
    - "/bin/sh"
    - "-c"
    - "while true; do date; sleep $TIME_FREQ;done > /opt/time/time-check.log"
```


## Task 4

1. Create a new Deployment called `nginx-deploy`, with:
    - One container called `nginx`
    - Image: `nginx:1.16`
    - `4` replicas
    - RollingUpdate strategy with:
        - `maxSurge=1`
        - `maxUnavailable=2`

2. Upgrade the Deployment to version `1.17`.

3. Once all pods are updated, undo the update and roll back to the previous version.

## Solution

Run the following command to create a manifest for deployment `nginx-deploy` and save it into a file:

`kubectl create deployment nginx-deploy --image=nginx:1.16 --replicas=4 --dry-run=client -oyaml > nginx-deploy.yaml`

and add the `strategy` field under the spec section as follows:

```yaml
strategy:
  rollingUpdate:
    maxSurge: 1
    maxUnavailable: 2
```

So final manifest file for deployment called `nginx-deploy` should looks like below:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx-deploy
  name: nginx-deploy
  namespace: default
spec:
  replicas: 4
  selector:
    matchLabels:
      app: nginx-deploy
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 2
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: nginx-deploy
    spec:
      containers:
      - image: nginx:1.16
        imagePullPolicy: IfNotPresent
        name: nginx
```

then run the `kubectl apply -f nginx-deploy.yaml` to create a deployment resource.

Now, upgrade the deployment's image version using the `kubectl set image` command:

`kubectl set image deployment nginx-deploy nginx=nginx:1.17`

Run the `kubectl rollout` command to undo the update and go back to the previous version:

`kubectl rollout undo deployment nginx-deploy`


## Task 5
Create a Redis Deployment

Create a deployment in the `default` namespace with the following specifications:

- Name: `redis`
- Image: `redis:alpine`
- Replicas: `1`
- Labels: `app=redis`
- CPU Request: `0.2` CPU (200m)
- Container Port: `6379`

Volumes:

1. An `emptyDir` volume named `data`, mounted at `/redis-master-data`.
2. A ConfigMap volume named `redis-config`, mounted at `/redis-master`.
    - The ConfigMap has already been created for you. Do not create it again.

Note: All resources should be created in the `default` namespace (unless specified otherwise).

## Solution

Solution manifest file to create a deployment `redis` as follows:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: redis
  name: redis
spec:
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      volumes:
      - name: data
        emptyDir: {}
      - name: redis-config
        configMap:
          name: redis-config
      containers:
      - image: redis:alpine
        name: redis
        volumeMounts:
        - mountPath: /redis-master-data
          name: data
        - mountPath: /redis-master
          name: redis-config
        ports:
        - containerPort: 6379
        resources:
          requests:
            cpu: "0.2"
```