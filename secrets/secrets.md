# Secrets
Secrets are used to store **sensitive data** such as passwords, tokens, or keys. Unlike ConfigMaps, they are **base64-encoded** and intended for confidential information.

1. Create Secrets
2. Inject into Pod

## **2 ways** to Create Secrets:
### 1. Imperative Approach
Used for quick creation from the command line.

```kubectl create secret generic {secret-name} --from-literal={key}={value}```

What it does:
* Creates a Secret with key-value pairs directly from the CLI.
* Values are automatically base64-encoded by Kubernetes

#### Example
```
kubectl create secret generic app-secret \
  --from-literal=DB_HOST=mysql \
  --from-literal=DB_USER=root \
  --from-literal=DB_PASSWORD=password
```

Explanation:
* app-secret -> name of the Secret
* Each --from-literal -> adds a key-value pair
* Values are stored securely (encoded) inside the cluster

#### From a file
```kubectl create secret generic {secret-name} --from-file={path-to-file}```

What it does:
* Creates a Secret from a file’s content.
* The file name becomes the key, and its content becomes the value.

#### Example
```kubectl create secret generic app-secret --from-file=app_secret.properties```
Explanation:
* Useful for large configs or multiple credentials
* Entire file content is stored as a single entry

### 2. Declarative Approach (Recommended)
Best for production, version control, and reproducibility.

#### Step 1: Define in YAML
*secret-data.yaml*
```
apiVersion: v1
kind: secret
metadata:
    name: app-secret
data:
    DB_Host: ncsinjXxcU1=
    DB_User: vcklefdmsPC34=
    DB_Password: jvencFS90=
```

Explanation:
* kind: Secret -> defines a Secret resource
* data -> must contain base64-encoded values
* Keys remain plain text, values are encoded


#### Step 2: Apply the configuration

```kubectl create -f secret-data.yaml```
Creates the Secret in the cluster.

*Comment*: Use ```kubectl apply -f``` for updates.

### Encoding & Decoding Secrets
#### Encode
```echo -n 'mysql' | base64```
Converts plain text into base64 format (required for YAML).

#### Decode 
```echo -n 'ncsinjXxcU1' | base64 --decode```
Converts base64 back to readable text.

### View Secrets

Lists all Secrets in the namespace:
```kubectl get secrets```

Shows metadata and structure (but NOT decoded values):
```kubectl describe secrets```

Displays full YAML including encoded values:
```kubectl get secret app-secret -o yaml```

### Using Secrets in Pods
#### Step 1: Inject as Environment Variables

*pod-definition.yaml*
```
apiVersion: v1 
kind: Pod 
metadata:
    name: simple-webapp-color
spec:
    containers:
        - name: simple-webapp-color
          image: simple-webapp-color
          ports:
            - containerPort: 8080
          envFrom:
            - secretRef:
                name: app-secret
```

*secret-data.yaml*
```
apiVersion: v1
kind: Secret
metadata:
    name: app-secret
data:
    DB_Host: ncsinjXxcU1=
    DB_User: vcklefdmsPC34=
    DB_Password: jvencFS90=
```

What happens:

All keys in app-secret become environment variables inside the container.

Example inside container:
```
DB_HOST=mysql
DB_USER=root
```



#### Step 2: Create the Pod
```kubectl create -f pod-definition.yaml```


### Using Secrets as Volumes
#### Step 1: Define volume
```
volumes:
- name: app-secret-volume
  secret:
    secretName: app-secret
```

Mounts the Secret as files inside the container.

#### Step 2: Inside the container
```$ ls /opt/app-secret-volumes```

Shows files:
```
DB_Host
DB_User
DB_Password
```

```$ cat /opt/app-secret-volumes/DB_Password```

Displays the decoded value.



# Scenarios

## Task 1: Create a Secret from Literal

Create a Secret named mysecret with:
- password=mypass

### Solution
```bash
kubectl create secret generic mysecret --from-literal=password=mypass
```

---

## Task 2: Create Secret from File

Create a file:

```bash
echo -n admin > username
```

Create the Secret:

```bash
kubectl create secret generic mysecret2 --from-file=username
```

---

## Task 3: Retrieve Secret Value

### Solution

```bash
kubectl get secret mysecret2 -o yaml
```

Decode value:

```bash
kubectl get secret mysecret2 -o jsonpath='{.data.username}' | base64 -d
```

---

## Task 4: Mount Secret as Volume

Create a Pod that mounts mysecret2 at /etc/foo

### Solution

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  volumes:
  - name: foo
    secret:
      secretName: mysecret2
  containers:
  - name: nginx
    image: nginx
    volumeMounts:
    - name: foo
      mountPath: /etc/foo
  restartPolicy: Never
```

Apply and verify:

```bash
kubectl apply -f pod.yaml
kubectl exec -it nginx -- ls /etc/foo
kubectl exec -it nginx -- cat /etc/foo/username
```

---

## Task 5: Use Secret as Environment Variable

Delete previous Pod and create a new one using env variable.

### Solution

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx
    env:
    - name: USERNAME
      valueFrom:
        secretKeyRef:
          name: mysecret2
          key: username
  restartPolicy: Never
```

Apply and verify:

```bash
kubectl apply -f pod.yaml
kubectl exec -it nginx -- env | grep USERNAME
```

---

## Task 6: Create Secret in Namespace

Create namespace and Secret:

```bash
kubectl create namespace secret-ops
kubectl create secret generic ext-service-secret \
  -n secret-ops \
  --from-literal=API_KEY=crwswRXCAcdsf3
```

---

## Task 7: Consume Secret as Environment Variable

Create Pod in namespace secret-ops:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: consumer
  namespace: secret-ops
spec:
  containers:
  - name: consumer
    image: nginx
    env:
    - name: API_KEY
      valueFrom:
        secretKeyRef:
          name: ext-service-secret
          key: API_KEY
  restartPolicy: Always
```

Apply and verify:

```bash
kubectl apply -f pod.yaml
kubectl exec -it -n secret-ops consumer -- /bin/sh
env
```

---

## Task 8: Create SSH Secret

Create SSH secret from file id_rsa.

### Solution

```bash
kubectl create secret generic my-secret \
  -n secret-ops \
  --type=kubernetes.io/ssh-auth \
  --from-file=ssh-privatekey=id_rsa
```

---

## Task 9: Mount Secret as Volume

Create Pod that mounts Secret at /var/app (read-only).

### Solution

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: consumer
  namespace: secret-ops
spec:
  containers:
  - name: consumer
    image: nginx
    volumeMounts:
    - name: secret-vol
      mountPath: /var/app
      readOnly: true
  volumes:
  - name: secret-vol
    secret:
      secretName: my-secret
  restartPolicy: Always
```

Apply and verify:

```bash
kubectl apply -f pod.yaml
kubectl exec -it -n secret-ops consumer -- /bin/sh
cat /var/app/ssh-privatekey
```