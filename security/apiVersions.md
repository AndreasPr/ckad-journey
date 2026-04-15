# API Versions in Kubernetes

---

## What are API Versions?

In Kubernetes, every resource belongs to:

- An **API Group**
- An **API Version**

Example:
```yaml
apiVersion: apps/v1
kind: Deployment
````

* `apps` → API Group
* `v1` → API Version

---

## Why Do API Versions Exist?

API versions allow Kubernetes to:

* Introduce new features safely
* Deprecate old functionality gradually
* Maintain backward compatibility

---

## Common API Version Levels

Kubernetes APIs typically evolve through stages:

### 1. Alpha (`v1alpha1`)

* Experimental
* Disabled by default
* May change or be removed

---

### 2. Beta (`v1beta1`)

* More stable than alpha
* Enabled by default (usually)
* May still change

---

### 3. Stable (`v1`)

* Production-ready
* Backward-compatible
* Long-term support

---

# API Groups and Versions

---

## Core Group

* Path: `/api`
* Example:

  * `/api/v1/pods`

---

## Named Groups

* Path: `/apis`
* Examples:

  * `/apis/apps/v1`
  * `/apis/batch/v1`

Each group can have **multiple versions**:

Example:

* apps/v1
* apps/v1beta1
* apps/v1beta2

---

# Preferred Version

---

## What is Preferred Version?

The **preferred version** is:

> The default version Kubernetes recommends for a given API group

---

## Characteristics

* Returned first in API discovery
* Used by default by `kubectl`
* Usually the **most stable version**

---

## Example

For the `apps` group:

* apps/v1beta1
* apps/v1beta2
* apps/v1  ← Preferred

---

## How to Identify Preferred Version

Run:

```bash
kubectl proxy
```

Then:

```bash
curl http://localhost:8001/apis/apps | jq
```

Look for:

```json
"preferredVersion": {
  "groupVersion": "apps/v1"
}
```

---

# Storage Version

---

## What is Storage Version?

The **storage version** is:

> The version in which objects are stored internally in etcd

---

## Important Notes

* Kubernetes converts objects:

  * From requested version → storage version
* Storage version is usually:

  * The **latest stable version**

---

## Example

Even if you create:

```yaml
apiVersion: apps/v1beta1
```

Kubernetes may store it internally as:

```
apps/v1
```

---

## Why This Matters

* Ensures consistency in etcd
* Simplifies upgrades and migrations
* Avoids storing multiple formats

---

# Preferred vs Storage Version

---

| Feature    | Preferred Version         | Storage Version           |
| ---------- | ------------------------- | ------------------------- |
| Purpose    | Client-facing default     | Internal storage format   |
| Used by    | kubectl / API clients     | etcd                      |
| Visibility | Visible via API discovery | Not directly visible      |
| Usually    | Stable version            | Same as preferred (often) |

---

# Relationship Between Them

* Clients interact with **preferred version**
* Kubernetes converts data to **storage version**
* Conversion is handled automatically by API server

---

# How to Identify API Versions

---

## List all API resources

```bash
kubectl api-resources
```

---

## List API versions

```bash
kubectl api-versions
```

---

## Inspect specific API group

```bash
kubectl proxy
curl http://localhost:8001/apis/apps
```

---

# How to Identify Storage Version

---

## Method 1: Check API Server Logs / Config

Storage version is defined internally in API server configuration.

---

## Method 2: Use `kubectl get --raw`

```bash
kubectl get --raw /apis/apps/v1
```

---

## Method 3 (Advanced): Storage Version API

```bash
kubectl get storageversion
```

(Requires appropriate permissions and newer Kubernetes versions)

---

# Enabling / Disabling API Versions

---

## Using kube-apiserver flag

You can control API versions using:

```bash
--runtime-config
```

---

## Example: Enable an Alpha API

```bash
--runtime-config=batch/v2alpha1=true
```

---

## Disable an API Version

```bash
--runtime-config=batch/v1beta1=false
```

---

## Where to Configure

Edit:

```
/etc/kubernetes/manifests/kube-apiserver.yaml
```

---

# Important Notes

---

## 1. Not All Versions Are Enabled

* Alpha APIs are usually **disabled by default**
* Must explicitly enable them

---

## 2. Deprecated APIs

* Older versions (e.g., `extensions/v1beta1`) are removed over time
* Always migrate to stable versions

---

## 3. Version Conversion

Kubernetes automatically:

* Converts between API versions
* Ensures compatibility across versions

---

# Summary

* API versions define how resources evolve over time
* Preferred version → default for clients
* Storage version → internal format in etcd
* Kubernetes handles conversion automatically
* Use `--runtime-config` to enable/disable versions

# Scenarios
## Scenario 1
Identify the short names of the `deployments`, `replicasets`, `cronjobs` and `customresourcedefinitions`.

## Solution
Run the command: `kubectl api-resources` and search them. 

Ex. `kubectl api-resources | grep deployments`

## Scenario 2
What is the patch version in the given Kubernetes API version?

Kubernetes API version - `1.22.2`

## Solution
In Kubernetes versions : `X.Y.Z`

Where `X` stands for major, `Y` stands for minor and `Z` stands for patch version.

## Scenario 3
Identify which API group a resource called `job` is part of?

## Solution
Run the command `kubectl explain job` and see the `API group` in the top of the line.

## Scenario 4
What is the preferred version for `authorization.k8s.io` api group?

## Solution
To identify the preferred version, run the following commands as follows :
```bash
root@controlplane:~# kubectl proxy 8001&
root@controlplane:~# curl localhost:8001/apis/authorization.k8s.io
```

Where `&` runs the command in the background and `kubectl proxy` command starts the proxy to the kubernetes API server.

## Scenario 5
Enable the `v1alpha1` version for `rbac.authorization.k8s.io` API group on the `controlplane` node.

Note: If you made a mistake in the config file could result in the API server being unavailable and can break the cluster.

## Solution

As a good practice, take a backup of that `apiserver` manifest file before going to make any changes.

In case, if anything happens due to misconfiguration you can replace it with the backup file.

```bash
root@controlplane:~# cp -v /etc/kubernetes/manifests/kube-apiserver.yaml /root/kube-apiserver.yaml.backup
```

Now, open up the `kube-apiserver` manifest file in the editor of your choice. It could be `vim` or `nano`.

```bash
root@controlplane:~# vi /etc/kubernetes/manifests/kube-apiserver.yaml
```

Add the `--runtime-config` flag in the `command` field as follows :

```bash
 - command:
    - kube-apiserver
    - --advertise-address=10.18.17.8
    - --allow-privileged=true
    - --authorization-mode=Node,RBAC
    - --client-ca-file=/etc/kubernetes/pki/ca.crt
    - --enable-admission-plugins=NodeRestriction
    - --enable-bootstrap-token-auth=true
    - --runtime-config=rbac.authorization.k8s.io/v1alpha1 --> This one 
```
  
After that `kubelet` will detect the new changes and will recreate the `apiserver` pod.

It may take some time.

```bash
root@controlplane:~# kubectl get po -n kube-system
```
Check the status of the `apiserver` pod. It should be in running condition.


## Scenario 6
Install the `kubectl convert` plugin on the `controlplane` node.

If unsure how to install then refer to the official k8s documentation page

## Solution

Download the latest release version from the `curl` command :

```bash
root@controlplane:~# curl -LO https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl-convert
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   154  100   154    0     0   6416      0 --:--:-- --:--:-- --:--:--  6416
100 60.7M  100 60.7M    0     0   142M      0 --:--:-- --:--:-- --:--:--  142M
```

then check the availability by `ls` command :

```bash
root@controlplane:~# pwd
/root
root@controlplane:~# ls
kubectl-convert  multi-pod.yaml  sample.yaml
```

Change the permission of the file and move to the `/usr/local/bin/` directory.

```bash
root@controlplane:~# pwd
/root
root@controlplane:~# chmod +x kubectl-convert 
root@controlplane:~# 
root@controlplane:~# mv kubectl-convert /usr/local/bin/kubectl-convert
root@controlplane:~# 
```

Use the `--help` option to see more option.
```bash
root@controlplane:~# kubectl-convert --help
```

If it'll show more options that means it's configured correctly if it'll give an error that means we haven't set up properly.

## Scenario 7
Ingress manifest file is already given under the `/root/` directory called `ingress-old.yaml`.

With help of the `kubectl convert` command, change the deprecated API version to the `networking.k8s.io/v1` and create the resource.

The `ingress-old.yaml` is:

```yaml
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: ingress-space
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - http:
      paths:
      - path: /video-service
        pathType: Prefix
        backend:
          serviceName: ingress-svc
          servicePort: 80
```


## Solution

Run the command `kubectl-convert` to change the deprecated API version as follows :-
```bash
root@controlplane:~# kubectl-convert -f ingress-old.yaml --output-version networking.k8s.io/v1

# store new changes into a file 
root@controlplane:~# kubectl-convert -f ingress-old.yaml --output-version networking.k8s.io/v1 > ingress-new.yaml
```

After changing the API version and storing into a file, use the `kubectl create -f` command to deploy the resource:

```bash
root@controlplane:~# kubectl create -f ingress-new.yaml
```

Inspect the `apiVersion` as follows :

```bash
root@controlplane:~# kubectl get ing ingress-space -oyaml | grep apiVersion
```

Note: Maybe you will not see the service and other resources mentioned in the `ingress` YAML on the `controlplane` node because we have to only deploy the `ingress` resource with the latest API version.


The `ingress-new.yaml` is:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
  name: ingress-space
spec:
  rules:
  - http:
      paths:
      - backend:
          service:
            name: ingress-svc
            port:
              number: 80
        path: /video-service
        pathType: Prefix
status:
  loadBalancer: {}
```