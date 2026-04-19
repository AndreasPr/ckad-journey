# Test 1

## Q1
Deploy a pod named `nginx-448839` using the `nginx:alpine image`.

## Solution
Use the command `kubectl run  nginx-448839 --image=nginx:alpine`


## Q2
Create a namespace named `apx-z993845`

## Solution
Run the command `kubectl create namespace apx-z993845`


## Q3
Create a new Deployment named `httpd-frontend` with `3` replicas using image `httpd:2.4-alpine`

## Solution

Command: `kubectl create deployment httpd-frontend --image=httpd:2.4-alpine --replicas=3`


## Q4
Deploy a messaging pod using the `redis:alpine` image with the labels set to `tier=msg`.


## Solution

Use the command `kubectl run messaging --image=redis:alpine -l tier=msg`


## Q5
A replicaset `rs-d33393` is created. However the pods are not coming up. Identify and fix the issue.

Once fixed, ensure the ReplicaSet has 4 `Ready` replicas.

## Solution

The image used for the replicaset should be `busybox` instead of `busyboxXXXXXXX`. Use `kubectl edit rs rs-d33393` to fix the image. Then delete all PODs to provision new ones with the new image.


## Q6
Create a service `messaging-service` to expose the `redis` deployment in the `marketing` namespace within the cluster on port `6379`.

## Solution
Run the command `kubectl expose deployment redis --port=6379 --name messaging-service --namespace marketing`


## Q7
Update the environment variable on the pod `webapp-color` to use a `green` background.

- Pod Name: `webapp-color`
- Label Name: `webapp-color`
- Env: `APP_COLOR=green`


## Solution
Set the environment variable `APP_COLOR` to `green`

Here is the solution YAML file:
```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: "2021-07-24T04:54:05Z"
  labels:
    name: webapp-color
  name: webapp-color
  namespace: default
spec:
  containers:
  - env:
    - name: APP_COLOR
      value: green
    image: andreascloud/webapp-color
    imagePullPolicy: Always
    name: webapp-color
```

Recreate the existing pod with the above YAML file. You can make use of the `replace` command like this:

`kubectl replace -f <above yaml file> --force`. This will delete the old pod and replace it with the new one with the configuration defined in the YAML file.


## Q8
Create a new ConfigMap named `cm-3392845`. Use the spec given on the below:

- ConfigName Name: `cm-3392845`
- Data: `DB_NAME=SQL3322`
- Data: `DB_HOST=sql322.mycompany.com`
- Data: `DB_PORT=3306`


## Solution
Use the command `kubectl create configmap cm-3392845 --from-literal=DB_NAME=SQL3322 --from-literal=DB_HOST=sql322.mycompany.com --from-literal=DB_PORT=3306`


## Q9
Create a new Secret named `db-secret-xxdf` with the data given (on the below):

- Secret Name: `db-secret-xxdf`
- Secret 1: `DB_Host=sql01`
- Secret 2: `DB_User=root`
- Secret 3: `DB_Password=password123`

## Solution

Run command `kubectl create secret generic db-secret-xxdf --from-literal=DB_Host=sql01 --from-literal=DB_User=root --from-literal=DB_Password=password123`


## Q10
Update pod `app-sec-kff3345` to run as Root user and with the `SYS_TIME` capability.

- Pod Name: `app-sec-kff3345`
- Image Name: `ubuntu`
- SecurityContext: Capability `SYS_TIME`


## Solution
Add `SYS_TIME` capability to the container's securityContext.
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-sec-kff3345
  namespace: default
spec:
  securityContext:
    runAsUser: 0
  containers:
  - command:
    - sleep
    - "4800"
    image: ubuntu
    name: ubuntu
    securityContext:
     capabilities:
        add: ["SYS_TIME"]
```


## Q11
Export the logs of the `e-com-1123` pod to the file `/opt/outputs/e-com-1123.logs`

## Solution
Run the command `kubectl logs e-com-1123 --namespace e-commerce > /opt/outputs/e-com-1123.logs`




## Q12
Create a `Persistent Volume` with the given specification.

- Volume Name: `pv-analytics`
- Storage: `100Mi`
- Access modes: `ReadWriteMany`
- Host Path: `/pv/data-analytics`

## Solution
The solution is provided below:

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-analytics
spec:
  capacity:
    storage: 100Mi
  volumeMode: Filesystem
  accessModes:
  - ReadWriteMany
  hostPath:
      path: /pv/data-analytics
```


## Q13
Create a `redis` deployment using the image `redis:alpine` with `1 replica` and label `app=redis`. Expose it via a ClusterIP service called `redis` on port `6379`. Create a new Ingress Type NetworkPolicy called `redis-access` which allows only the pods with label `access=redis` to access the deployment.

## Solution
To create deployment:

`kubectl create deployment redis --image=redis:alpine --replicas=1`

To expose the deployment using ClusterIP:

`kubectl expose deployment redis --name=redis --port=6379 --target-port=6379`

To create ingress rule:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: redis-access
  namespace: default
spec:
  podSelector:
    matchLabels:
       app: redis
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          access: redis
    ports:
     - protocol: TCP
       port: 6379
```


## Q14

Create a Pod called `sega` with two containers:
- Container 1: Name `tails` with image `busybox` and command: `sleep 3600`.
- Container 2: Name `sonic` with image `nginx` and Environment variable: `NGINX_PORT` with the value `8080`.

## Solution

The pod yaml file should be:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sega
spec:
  containers:
  - image: busybox
    name: tails
    command:
    - sleep
    - "3600"
  - image: nginx
    name: sonic
    env:
    - name: NGINX_PORT
      value: "8080"
```