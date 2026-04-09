# Service Accounts in Kubernetes

Service Accounts are used by **applications (Pods)** to interact with the Kubernetes API securely.

---

## Types of Accounts

1. **User Accounts**
   - Used by humans (e.g., admins, developers)
   - Authenticated via certificates, OIDC, etc.

2. **Service Accounts**
   - Used by applications (Pods)
   - Provide identity inside the cluster

---

## Creating and Managing Service Accounts

### Create a ServiceAccount named dashboard-sa in the current namespace.
```kubectl create serviceaccount dashboard-sa```


### List Service Accounts
```kubectl get serviceaccount```

### Describe a Service Account
```kubectl describe serviceaccount dashboard-sa```

Shows:
* Associated tokens
* Secrets (pre-v1.24 behavior)
* Metadata

## Service Account Tokens (Pre v1.24 Behavior)
* Kubernetes automatically created a Secret containing a token.
* This token was:
    * Mounted into Pods
    * Used for API authentication
    * Non-expiring (security risk)

```kubectl describe secret dashboard-sa-token-kbbdm```

* Each namespace has its own default service account
* The secret token is located in a volume of a pod

### Inspect Token Inside a Pod
```kubectl exec -it my-kubernetes-dashboard ls /var/run/secrets/kubernetes.io/serviceaccount```
```kubectl exec -it my-kubernetes-dashboard cat /var/run/secrets/kubernetes.io/serviceaccount/token```
* This token allows the Pod to communicate with the Kubernetes API.



## Important Behavior
* Every namespace has a default ServiceAccount
* If not specified -> Kubernetes automatically assigns it
* The token is mounted as a volume inside the Pod


## Using a Custom Service Account in a Pod

### Example
```
apiVersion: v1
kind: Pod
metadata:
  name: my-kubernetes-dashboard
spec:
  serviceAccountName: dashboard-sa
  containers:
  - name: my-kubernetes-dashboard
    image: my-kubernetes-dashboard
```

* This Pod will use dashboard-sa instead of the default.

**Important Limitation**
* You cannot modify the ServiceAccount of a running Pod
* You must delete and recreate the Pod

**Exception:**
* In a Deployment, updating the ServiceAccount triggers a new rollout

## Disable Automatic Token Mounting

By default, Kubernetes mounts a token into every Pod.

To disable this:

```
apiVersion: v1
kind: Pod
metadata:
  name: my-kubernetes-dashboard
spec:
  automountServiceAccountToken: false
  containers:
  - name: my-kubernetes-dashboard
    image: my-kubernetes-dashboard
```

* Improves security when API access is not needed.



# UPDATE: Version v1.24


# Kubernetes v1.24+ Changes (IMPORTANT)
## KEP-2799: Token Behavior Change

### Starting from v1.24:

* Kubernetes NO LONGER auto-creates Secret-based tokens
* Tokens are now:
    * Short-lived
    * Generated on demand
    * More secure

#### Generate a Token (Recommended Way)
```kubectl create serviceaccount dashboard-sa```

after that, we should generate the token:
```kubectl create token dashboard-sa```

* This uses the TokenRequest API


# TokenRequest API (Recommended)

**Benefits**
* Tokens are temporary (time-bound)
* Automatically expire (default ~1 hour, configurable)
* Reduce risk of credential leakage

#### Example:
* Token expires after a short period
* Must be refreshed if needed again

### Legacy Approach (Not Recommended)

You can still create non-expiring tokens manually:

```
apiVersion: v1
kind: Secret
type: kubernetes.io/service-account-token
metadata:
  name: mysecretname
  annotations:
    kubernetes.io/service-account.name: dashboard-sa
```

```kubectl create -f secret-definition.yaml```


**Important Notes**
* Create the ServiceAccount before creating this Secret
* This creates a long-lived token (security risk)

## When to Use Legacy Tokens?

Only if:

* You cannot use TokenRequest API
* You accept the risk of non-expiring credentials



# Exercise
Modify the existing ServiceAccount ```dashboard-sa``` in the ```default``` namespace to disable automatic mounting of API credentials.

Then, modify the existing ```web-dashboard``` Deployment to inject (mount) a ServiceAccount token at:

```/var/run/secrets/kubernetes.io/serviceaccount/token```

Use a projected volume named ```token``` to inject the ServiceAccount token, and ensure it is mounted read-only.

PS: The Deployment manifest file can be found at: ```~/web-dashboard/deployment.yaml```

# Solution
This task has two parts: disabling automatic token mounting on the ServiceAccount, and configuring a projected volume on the Deployment.

## Part 1: Disable automountServiceAccountToken on the ServiceAccount

1. Edit the ServiceAccount:
```kubectl edit sa dashboard-sa```

2. Add ```automountServiceAccountToken: false``` at the top level:

```apiVersion: v1
automountServiceAccountToken: false
kind: ServiceAccount
metadata:
  name: dashboard-sa
  namespace: 
```

3. Save and close the editor.

Alternatively, use ```kubectl patch```:

```kubectl patch sa dashboard-sa -p '{"automountServiceAccountToken": false}'```


## Part 2: Configure projected volume on the Deployment

1. Edit the deployment manifest:
```vim ~/web-dashboard/deployment.yaml```

2. Add the following changes to ```spec.template.spec```:

* ```automountServiceAccountToken: false``` to disable default token mounting
* A projected volume named ```token``` with a ```serviceAccountToken``` source
* A volumeMount in the container to mount the token at ```/var/run/secrets/kubernetes.io/serviceaccount/``` with ```readOnly: true```

3. The final manifest should look like:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-dashboard
  namespace: default
  labels:
    name: web-dashboard
spec:
  replicas: 1
  selector:
    matchLabels:
      name: web-dashboard
  template:
    metadata:
      labels:
        name: web-dashboard
    spec:
      containers:
      - name: web-dashboard
        image: gcr.io/andreas/customimage/my-kubernetes-dashboard
        ports:
        - containerPort: 8080
        env:
        - name: PYTHONUNBUFFERED
          value: "1"
        volumeMounts:
        - mountPath: /var/run/secrets/kubernetes.io/serviceaccount/
          name: token
          readOnly: true
      serviceAccountName: dashboard-sa
      automountServiceAccountToken: false
      volumes:
      - name: token
        projected:
          sources:
          - serviceAccountToken:
              path: token
```

4. Apply the manifest:
```kubectl apply -f ~/web-dashboard/deployment.yaml```

5. Verify the pod is running:
```kubectl get pods```


The ```web-dashboard``` Deployment is now configured with a manually mounted ServiceAccount token using a projected volume.
This approach is more secure than the default automatic token mounting because it gives you explicit control over which pods receive API credentials.

Verify the token is mounted at the expected path:

```kubectl exec $(kubectl get pod -l name=web-dashboard -o jsonpath='{.items[0].metadata.name}') -- ls /var/run/secrets/kubernetes.io/serviceaccount/```