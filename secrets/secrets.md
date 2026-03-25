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