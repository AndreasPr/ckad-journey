# What is kubeconfig?

---

## Overview

`kubeconfig` is a configuration file used by `kubectl` to **connect to Kubernetes clusters and authenticate users**.

Instead of repeatedly passing connection and authentication parameters via CLI flags, kubeconfig allows you to **store all this information in one place**.

---

## The Problem kubeconfig Solves

When accessing the Kubernetes API directly, you must provide authentication details manually.

Example using `curl`:

```
curl https://my-kube-playground:6443/api/v1/pods \
  --key admin.key \
  --cert admin.crt \
  --cacert ca.crt
````

Or using `kubectl`:

```
kubectl get pods \
  --server my-kube-playground:6443 \
  --client-key admin.key \
  --client-certificate admin.crt \
  --certificate-authority ca.crt
```

This becomes tedious and error-prone.

---

## Solution: kubeconfig File

Instead of passing parameters every time, you define them once in a kubeconfig file:

```
kubectl get pods --kubeconfig=config
```

By default, Kubernetes looks for the kubeconfig file at:

```
$HOME/.kube/config
```

---

## What kubeconfig Contains

The kubeconfig file organizes configuration into **three main sections**:

The `config` file has the following sections:
1. `clusters` for different environments (dev, prod), different organizations, or different cloud providers(google), etc
2. `users` are user accounts with which you have access to these clusters, for example the admin user, dev user, prod user, etc
3. `contexts` define which user account will be used to access which cluster, for example admin@production, dev@google


---

### 1. Clusters

Defines the Kubernetes clusters you can access.

```
clusters:
- name: my-kube-playground
  cluster:
    server: https://my-kube-playground:6443
    certificate-authority: ca.crt
```

* `server`: API server endpoint
* `certificate-authority`: CA certificate used to verify the server

---

### 2. Users

Defines authentication credentials.

```
users:
- name: my-kube-admin
  user:
    client-certificate: admin.crt
    client-key: admin.key
```

* Represents identities (admin, dev, etc.)
* Uses certificates, tokens, or other auth mechanisms

---

### 3. Contexts

Links a **user to a cluster**.

```
contexts:
- name: my-kube-admin@my-kube-playground
  context:
    cluster: my-kube-playground
    user: my-kube-admin
```

A context defines:

* Which cluster to connect to
* Which user credentials to use

---

## Current Context

The active context is defined by:

```
current-context: my-kube-admin@my-kube-playground
```

This determines the default cluster and user for all `kubectl` commands.

---

## Important Concept

kubeconfig does NOT:

* Create users
* Grant permissions
* Modify cluster security

It simply **uses existing credentials and access rights**.

---

## Full Example

```
apiVersion: v1
kind: Config

current-context: dev-user@google

clusters:
- name: my-kube-playground
  cluster:
    server: https://my-kube-playground:6443
    certificate-authority: ca.crt

contexts:
- name: my-kube-admin@my-kube-playground
  context:
    cluster: my-kube-playground
    user: my-kube-admin

users:
- name: my-kube-admin
  user:
    client-certificate: admin.crt
    client-key: admin.key
```

---

## Common Commands

### View kubeconfig

```
kubectl config view
```

---

### Use a Specific kubeconfig File

```
kubectl config view --kubeconfig=my-custom-config
```

---

### Switch Context

```
kubectl config use-context prod-user@production
```

---

## Context with Namespace

You can define a default namespace per context:

```
contexts:
- name: admin@production
  context:
    cluster: production
    user: admin
    namespace: finance
```

**Now all commands run in the `finance` namespace by default.**

---

## Certificate Configuration

### Using File Paths

```
clusters:
- name: my-kube-playground
  cluster:
    server: https://my-kube-playground:6443
    certificate-authority: /etc/kubernetes/pki/ca.crt

users:
- name: my-kube-admin
  user:
    client-certificate: /etc/kubernetes/pki/users/admin.crt
    client-key: /etc/kubernetes/pki/users/admin.key
```

---

### Using Embedded Certificates (Base64)

Instead of referencing files, you can embed certificate data:

```
clusters:
- name: my-kube-playground
  cluster:
    server: https://my-kube-playground:6443
    certificate-authority-data: <base64-encoded-cert>
```

---

### Encode Certificate

```cat ca.crt | base64```

---

### Decode Certificate

```echo <base64-string> | base64 --decode```

---

## Summary

* kubeconfig stores cluster access configuration
* Eliminates the need to pass credentials manually
* Organizes access using clusters, users, and contexts
* Does not manage authentication or authorization itself
* Enables easy switching between environments



# Scenarios
## Scenario 1
We don't want to specify the kubeconfig file option on each kubectl command.
Set the my-kube-config file as the default kubeconfig file and make it persistent across all sessions without overwriting the existing ~/.kube/config. Ensure any configuration changes persist across reboots and new shell sessions.
Note: Don't forget to source the configuration file to take effect in the existing session. Example:
`source ~/.bashrc`

## Solution
Add the `my-kube-config` file to the `KUBECONFIG` environment variable.

Open your shell configuration file:
`vi ~/.bashrc`

Add one of these lines to export the variable:
```
export KUBECONFIG=/root/my-kube-config
# OR
export KUBECONFIG=~/my-kube-config
# OR
export KUBECONFIG=$HOME/my-kube-config
```

Apply the Changes:

Reload the shell configuration to apply the changes in the current session:

`source ~/.bashrc`

## Scenario 2
With the current-context set to `research`, we are trying to access the cluster. However something seems to be wrong. Identify and fix the issue.


Try running the `kubectl get pods` command and look for the error. All users certificates are stored at `/etc/kubernetes/pki/users`.
## Solution

Before the fix , You will notice you cant get the pods in the cluster :
```
controlplane ~ ➜  kubectl get pods
error: unable to read client-cert /etc/kubernetes/pki/users/dev-user/developer-user.crt for dev-user due to open /etc/kubernetes/pki/users/dev-user/developer-user.crt: no such file or directory
```

Solution steps:

1. Identify the Current Certificate Path
```
kubectl config view --kubeconfig=/root/my-kube-config | grep -A5 "name: dev-user"
```

2. Verify the Actual Certificate Location
```
ls -l /etc/kubernetes/pki/users/dev-user/
```

3. Edit the Kubeconfig File
```
kubectl config set-credentials dev-user \
  --client-certificate=/etc/kubernetes/pki/users/dev-user/dev-user.crt \
  --client-key=/etc/kubernetes/pki/users/dev-user/dev-user.key \
  --kubeconfig=/root/my-kube-config
```

4. Test Cluster Access
```
controlplane ~ ➜  kubectl get pods
No resources found in default namespace.
```