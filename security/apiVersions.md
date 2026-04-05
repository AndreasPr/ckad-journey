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

