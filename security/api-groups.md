# Kubernetes API Groups

---

## What are API Groups?

In Kubernetes, **API Groups** are a way to organize and categorize the Kubernetes API into logical sections.

They help:
- Structure the API in a scalable way  
- Group related resources together  
- Enable independent evolution of features  

---

## High-Level Structure

Kubernetes APIs are divided into two main categories:

---

### 1. Core API Group (`/api`)

This is the **original and fundamental API group**.

Endpoint:
```
/api/v1
```

---

#### Contains Core Resources

* Pods
* Services
* Namespaces
* Nodes
* Secrets
* ConfigMaps
* ReplicationControllers

These are considered the **basic building blocks** of Kubernetes.

---

### 2. Named API Groups (`/apis`)

This is the **extended and modern API structure**.

Endpoint:

```
/apis/<group-name>/<version>
```

---

#### Examples of Named API Groups

* `/apis/apps`
* `/apis/extensions`
* `/apis/networking.k8s.io`
* `/apis/storage.k8s.io`
* `/apis/authentication.k8s.io`
* `/apis/certificates.k8s.io`

These groups organize newer and more advanced features.

---

## API Groups → Resources → Verbs

Kubernetes API is structured hierarchically:

```
API Group → Resource → Verbs (Actions)
```

---

### Example: `/apps` API Group

Resources under `/apps`:

* deployments
* replicasets
* statefulsets

---

### Example: `/certificates.k8s.io` API Group

Resources:

* certificatesigningrequests

---

## What are Resources?

Resources are the **objects you interact with in Kubernetes**.

Examples:

* Pods
* Deployments
* Services

Each resource belongs to an API group.

---

## What are Verbs?

Verbs define **what actions you can perform on a resource**.

Common verbs:

* get → retrieve a resource
* list → list multiple resources
* create → create a new resource
* update → modify a resource
* delete → remove a resource
* watch → observe changes in real time

---

## Example API Path

```
/apis/apps/v1/deployments
```

Breakdown:

* `/apis` → named API groups
* `/apps` → API group
* `/v1` → API version
* `/deployments` → resource

---

## Exploring the API

---

### Get API Versions

```
curl https://localhost:6443 -k
```

---

### List API Groups

```
curl https://localhost:6443/apis -k
```

---

### Filter API Group Names

```
curl https://localhost:6443/apis -k | grep "name"
```

---

## Authenticated API Access

```
curl https://localhost:6443 \
  --key admin.key \
  --cert admin.crt \
  --cacert ca.crt
```

---

## Easier Access with kubectl proxy

Instead of manually providing certificates:

```
kubectl proxy
```

This:

* Starts a local proxy (default: port 8001)
* Uses credentials from kubeconfig
* Handles authentication automatically

---

Therefore:

```
curl http://localhost:8001 -k
```

---

## Why API Groups Matter

* Enable modular API design
* Allow independent versioning
* Prevent conflicts between resources
* Support extensibility (CRDs, custom APIs)

---

## Summary

* Kubernetes APIs are divided into **Core** and **Named** groups
* Core group contains fundamental resources
* Named groups organize modern and extended features
* Resources represent objects (pods, deployments, etc.)
* Verbs define allowed operations (get, create, delete, etc.)
* API groups provide structure, scalability, and flexibility
