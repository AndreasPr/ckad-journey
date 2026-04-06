# Helm in Kubernetes

## What is Helm?

**Helm** is a package manager for Kubernetes that helps you **define, install, and manage applications** on a Kubernetes cluster.

Think of it like:

* `apt` for Ubuntu
* `yum` for CentOS
* `npm` for Node.js

But specifically for **Kubernetes applications**.

---

## Why Helm?

Deploying applications in Kubernetes often requires:

* Multiple YAML files (Deployments, Services, ConfigMaps, Secrets, etc.)
* Environment-specific configurations
* Versioning and upgrades

Helm solves this by:

* Packaging all resources into a **Chart**
* Supporting **templating**
* Managing **releases and versions**

---

## Core Concepts

### 1. Chart

A **Chart** is a package of Kubernetes resources.

It contains:

* YAML templates
* Default configuration values
* Metadata

Example structure:

```
mychart/
  Chart.yaml
  values.yaml
  templates/
    deployment.yaml
    service.yaml
```

---

### 2. Release

A **Release** is a deployed instance of a Chart in a cluster.

Example:

* Chart: `wordpress`
* Release: `my-blog`

You can install the same chart multiple times with different configurations.

---

### 3. Values

Values are configuration inputs for templates.

Example:

```yaml
replicaCount: 3
image:
  tag: "1.2.0"
```

Override them during install:

```bash
helm install myapp ./chart -f custom-values.yaml
```

---

### 4. Templates

Helm uses templating (Go templates) to generate Kubernetes manifests dynamically.

Example:

```yaml
replicas: {{ .Values.replicaCount }}
```

---

## Basic Helm Commands

### Install an application

```bash
helm install wordpress
```

Better example:

```bash
helm install my-wordpress bitnami/wordpress
```

---

### Upgrade a release

```bash
helm upgrade wordpress
```

Example:

```bash
helm upgrade my-wordpress bitnami/wordpress --set replicaCount=5
```

---

### Rollback a release

```bash
helm rollback wordpress
```

Example:

```bash
helm rollback my-wordpress 1
```

---

### Uninstall a release

```bash
helm uninstall wordpress
```

Removes:

* All Kubernetes resources created by the release
* Release metadata

---

## Lifecycle of a Helm Release

1. Install → creates resources
2. Upgrade → modifies resources
3. Rollback → reverts to previous version
4. Uninstall → deletes everything

---

## Helm Repositories

Helm charts are stored in repositories.

Example:

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
```

Search charts:

```bash
helm search repo wordpress
```

---

## Advantages of Helm

### 1. Simplifies Deployments

Instead of multiple YAML files:

```bash
helm install myapp
```

---

### 2. Reusability

* Same chart across environments (dev, staging, prod)

---

### 3. Version Control

* Track changes
* Rollback easily

---

### 4. Parameterization

* Customize deployments without editing YAML

---

### 5. Release Management

* Multiple instances of same app
* Easy upgrades and rollbacks

---

## Helm vs kubectl

| Feature    | kubectl | Helm |
| ---------- | ------- | ---- |
| Apply YAML | Yes     | Yes  |
| Templating | No      | Yes  |
| Versioning | No      | Yes  |
| Rollback   | No      | Yes  |
| Packaging  | No      | Yes  |

---

## Helm vs Operators

| Feature              | Helm             | Operator            |
| -------------------- | ---------------- | ------------------- |
| Deployment           | Yes              | Yes                 |
| Lifecycle automation | Limited          | Advanced            |
| Self-healing         | No               | Yes                 |
| Logic                | Static templates | Dynamic controllers |

---

## Useful Questions

### 1. What problem does Helm solve?

Helm simplifies Kubernetes deployments by packaging resources, enabling templating, and managing versions/releases.

---

### 2. What is the difference between a Chart and a Release?

* **Chart** → Blueprint/template
* **Release** → Running instance of that chart

---

### 3. How does Helm support rollback?

Helm keeps revision history of releases and allows reverting to previous versions using:

```bash
helm rollback <release> <revision>
```

---

### 4. What is `values.yaml`?

A file that defines default configuration values used by templates.

---

### 5. How do you override values?

* `--set key=value`
* `-f custom-values.yaml`

---

### 6. What happens during `helm upgrade`?

* Helm generates new manifests
* Compares with existing resources
* Applies changes (like `kubectl apply`)

---

### 7. Is Helm stateful?

Yes. Helm stores release metadata (usually in Kubernetes secrets).

---

## When to Use Helm

Use Helm when:

* You deploy complex applications
* You need reusable templates
* You want versioning and rollback
* You manage multiple environments

---

## When NOT to Use Helm

Avoid Helm if:

* Very simple deployments
* You need complex runtime logic → use Operators instead

---

## Summary

Helm is:

* A **package manager for Kubernetes**
* Uses **Charts** to define applications
* Manages **Releases** for deployment lifecycle
* Enables **templating, versioning, and rollback**



## Commands

### Identify the version of helm installed on the cluster.
`helm version`

### Command line flag used to enable verbose output?

`helm --debug`

### Identify the name of the Operating system installed.

Run the command `cat /etc/*release*` or `cat /etc/os-release` and identify the name of the operating system.