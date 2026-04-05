# API Deprecations in Kubernetes

---

## What are API Deprecations?

**API deprecation** is the process of marking an API version as **obsolete** and planning its eventual removal.

Instead of breaking users immediately, Kubernetes:
1. Marks an API as **deprecated**
2. Continues supporting it for a defined period
3. Eventually **removes** it in a future release

---

## Why Do We Need Multiple API Versions?

Kubernetes evolves rapidly. Supporting multiple versions allows:

- Backward compatibility for existing workloads  
- Safe introduction of new features  
- Gradual migration for users  

---

## How Many Versions Should You Support?

Best practice:

- Use **only the latest stable version (v1)** whenever possible  
- Avoid:
  - Alpha versions (unstable)  
  - Deprecated versions  

---

## When Can an API Version Be Removed?

Kubernetes follows a strict **API Deprecation Policy** to ensure stability.

---

# API Deprecation Policy Rules

---

## Rule #1  
### API elements may only be removed by incrementing the version of the API group

---

### Meaning

You **cannot remove fields or resources** from an existing API version.

Instead:
- You must create a **new version** of the API group

---

### Example

Bad (NOT allowed):
```yaml
apiVersion: apps/v1
# removing a field here ❌
````

Correct approach:

* Keep `apps/v1` unchanged
* Introduce:

```yaml
apiVersion: apps/v2
```

---

### Why?

* Prevents breaking existing clients
* Ensures stability for production workloads

---

## Rule #2

### API objects must round-trip between versions without information loss

---

### Meaning

If an object is:

* Converted from version A → version B → version A

It must remain **unchanged**

---

### Example

1. Create a Deployment in:

```yaml
apiVersion: apps/v1beta1
```

2. Kubernetes converts it to:

```yaml
apps/v1
```

3. Convert back → must preserve all fields

---

### Exception

* If a resource **does not exist** in a version, it may be excluded

---

### Why?

* Ensures compatibility between tools and clients
* Prevents silent data loss

---

## Rule #3

### An API version cannot be deprecated until a newer stable version exists

---

### Meaning

You cannot deprecate:

* `v1beta1`

Unless:

* A newer version exists (e.g., `v1`)

---

### Example

* `extensions/v1beta1` Deployments were deprecated
* Only AFTER `apps/v1` became stable

---

### Why?

* Users must always have a **migration path**

---

## Rule #4a

### Minimum support duration after deprecation

---

Once deprecated, APIs must remain available for:

| Version Type | Minimum Support                               |
| ------------ | --------------------------------------------- |
| GA (v1)      | 12 months OR 3 releases (whichever is longer) |
| Beta         | 9 months OR 3 releases                        |
| Alpha        | No guarantee                                  |

---

### Example

* `apps/v1beta1` deprecated in Kubernetes 1.16
* Removed in Kubernetes 1.19

(3 release rule applied)

---

### Why?

* Gives users time to migrate
* Avoids breaking production systems

---

## Rule #4b

### Preferred & Storage version cannot change immediately

---

### Meaning

When introducing a new version:

* Kubernetes must support:

  * OLD version
  * NEW version

Before:

* Making the new version the **preferred** or **storage version**

---

### Example

1. Introduce:

   * `apps/v2`
   * Keep `apps/v1`

2. Release both versions

3. In a later release:

   * Promote `apps/v2` to:

     * Preferred version
     * Storage version

---

### Why?

* Ensures safe migration
* Avoids breaking clients relying on old versions

---

# Summary of Policy Rules

---

| Rule | Purpose                        |
| ---- | ------------------------------ |
| #1   | Prevent breaking changes       |
| #2   | Ensure data consistency        |
| #3   | Guarantee migration path       |
| #4a  | Provide deprecation window     |
| #4b  | Ensure safe version transition |

---

# kubectl convert

---

## What is it?

`kubectl convert` helps:

> Convert Kubernetes manifests from one API version to another

---

## Example

```kubectl convert -f nginx.yaml --output-version apps/v1```

---

## What it does

* Reads your YAML file
* Converts it to a newer API version
* Outputs updated YAML

---

## Example Scenario

Old Deployment:

```yaml
apiVersion: extensions/v1beta1
```

Convert to:

```yaml
apiVersion: apps/v1
```

---

## Why Use It?

* Helps migrate deprecated APIs
* Ensures compatibility with newer clusters

---

## Important Notes

* `kubectl convert` is:

  * Not included by default
  * Available as a **plugin**

---

## Installation (example)

```kubectl krew install convert```

---

## Limitations

* May not handle all edge cases
* Manual validation still required

---

# Best Practices

---

* Always use:

  * Latest stable API versions (`v1`)
* Regularly check:

  * Deprecated APIs
* Test upgrades before production
* Use:

  * `kubectl convert` for migrations

---

# Summary

* API deprecation ensures safe evolution of Kubernetes
* Multiple versions provide backward compatibility
* Strict rules prevent breaking changes
* Always migrate before removal deadlines



# Scenarios
## Scenario 1:
What is the patch version in the given Kubernetes API version?
Kubernetes API version - 1.22.2

## Solution
In Kubernetes versions : X.Y.Z

Where X stands for major, Y stands for minor and Z stands for patch version.


## Scenario 2:
Identify which API group a resource called job is part of?

## Solution
Run the command kubectl explain job and see the API group in the top of the line.


## Scenario 3:
What is the preferred version for `authorization.k8s.io` api group?

## Solution
To identify the preferred version, run the following commands as follows :-

root@controlplane:~# `kubectl proxy 8001&`
root@controlplane:~# `curl localhost:8001/apis/authorization.k8s.io`

Where `&` runs the command in the background and `kubectl proxy` command starts the proxy to the kubernetes API server.


## Scenario 4:
Enable the `v1alpha1` version for `rbac.authorization.k8s.io` API group on the `controlplane` node.
Note: If you made a mistake in the config file could result in the API server being unavailable and can break the cluster.
runtime-config option added?
kube-apiserver pod is running?

## Solution
As a good practice, take a backup of that `apiserver` manifest file before going to make any changes.

In case, if anything happens due to misconfiguration you can replace it with the backup file.

`root@controlplane:~# cp -v /etc/kubernetes/manifests/kube-apiserver.yaml /root/kube-apiserver.yaml.backup`
Now, open up the kube-apiserver manifest file in the editor of your choice. It could be vim or nano.

```root@controlplane:~# vi /etc/kubernetes/manifests/kube-apiserver.yaml```

Add the `--runtime-config` flag in the `command` field as follows :-
```yaml
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

After that `kubelet` will detect the new changes and will recreate the apiserver pod.

It may take some time.

`root@controlplane:~# kubectl get po -n kube-system`
Check the status of the apiserver pod. It should be in running condition.


## Scenario 5:
Install the `kubectl convert` plugin on the `controlplane` node.

## Solution

Download the latest release version from the curl command :-

`root@controlplane:~# curl -LO https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl-convert`
```
% Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   154  100   154    0     0   6416      0 --:--:-- --:--:-- --:--:--  6416
100 60.7M  100 60.7M    0     0   142M      0 --:--:-- --:--:-- --:--:--  142M
```

then check the availability by `ls` command :-

```
root@controlplane:~# pwd
/root
root@controlplane:~# ls
kubectl-convert  multi-pod.yaml  sample.yaml
```

Change the permission of the file and move to the `/usr/local/bin/` directory.
```
root@controlplane:~# pwd
/root
root@controlplane:~# chmod +x kubectl-convert 
root@controlplane:~# 
root@controlplane:~# mv kubectl-convert /usr/local/bin/kubectl-convert
root@controlplane:~# 
```

Use the `--help` option to see more option.

`root@controlplane:~# kubectl-convert --help`

If it'll show more options that means it's configured correctly if it'll give an error that means we haven't set up properly.


## Scenario 6:
Ingress manifest file is already given under the `/root/` directory called `ingress-old.yaml`.

With help of the `kubectl convert` command, change the deprecated API version to the `networking.k8s.io/v1` and create the resource.

## Solution
Run the command `kubectl-convert` to change the deprecated API version as follows :-
```
root@controlplane:~# kubectl-convert -f ingress-old.yaml --output-version networking.k8s.io/v1

# store new changes into a file 
root@controlplane:~# kubectl-convert -f ingress-old.yaml --output-version networking.k8s.io/v1 > ingress-new.yaml
```

After changing the API version and storing into a file, use the `kubectl create -f` command to deploy the resource :-

```root@controlplane:~# kubectl create -f ingress-new.yaml```

Inspect the `apiVersion` as follows :-

```root@controlplane:~# kubectl get ing ingress-space -o yaml | grep apiVersion```

Note: Maybe you will not see the service and other resources mentioned in the `ingress` YAML on the `controlplane` node because we have to only deploy the `ingress` resource with the latest API version.