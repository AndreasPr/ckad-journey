# Imperative vs Declarative in Kubernetes

In Kubernetes, we primarily work in a **declarative manner** using YAML configuration files. However, **imperative commands** are extremely useful when:

* Quickly creating resources
* Testing configurations
* Generating YAML templates for further customization

---

# Key Flags for Imperative Commands

## 1. `--dry-run=client`

* Prevents the resource from being created
* Validates the command locally
* Useful for testing commands safely

---

## 2. `-o yaml`

* Outputs the resource definition in YAML format
* Allows you to inspect or modify before applying

---

## Combined Usage

These flags are commonly used together to generate YAML templates:

```bash
kubectl run nginx --image=nginx --dry-run=client -o yaml > nginx-pod.yaml
```

This generates a Pod definition file without creating the resource.

---

# POD Commands

## Create a Pod

```bash
kubectl run nginx --image=nginx
```

---

## Generate Pod YAML (without creating)

```bash
kubectl run nginx --image=nginx --dry-run=client -o yaml
```

---

## Create Pod with Labels

```bash
kubectl run redis --image=redis:alpine --labels="tier=db"
```

---

## Create Pod with Exposed Port

```bash
kubectl run httpd --image=httpd:alpine --port=80 --expose=true
```

* Creates a Pod
* Automatically creates a **ClusterIP Service**

---

# Deployment Commands

## Create a Deployment

```bash
kubectl create deployment nginx --image=nginx
```

---

## Generate Deployment YAML

```bash
kubectl create deployment nginx --image=nginx --dry-run=client -o yaml
```

---

## Create Deployment with Replicas

```bash
kubectl create deployment nginx --image=nginx --replicas=4
```

---

## Scale Deployment

```bash
kubectl scale deployment nginx --replicas=4
```

---

# Recommended Workflow

A best-practice approach:

1. Generate YAML:

```bash
kubectl create deployment nginx --image=nginx --dry-run=client -o yaml > nginx-deployment.yaml
```

2. Modify the file (e.g., replicas, resources, labels)

3. Apply it:

```bash
kubectl apply -f nginx-deployment.yaml
```

This combines:

* Speed (no need to write YAML from scratch)
* Flexibility (full control over configuration)

---

# Service Commands

## Create ClusterIP Service (Recommended)

```bash
kubectl expose pod redis --port=6379 --name=redis-service --dry-run=client -o yaml
```

### Advantages

* Automatically uses Pod labels as selectors

---

## Alternative: Create ClusterIP Service

```bash
kubectl create service clusterip redis --tcp=6379:6379 --dry-run=client -o yaml
```

### Limitations

* Uses default selector (`app=redis`)
* Does not match actual Pod labels automatically
* Requires manual correction

---

## Create NodePort Service (Recommended)

```bash
kubectl expose pod nginx --port=80 --name=nginx-service --type=NodePort --dry-run=client -o yaml
```

### Advantages

* Correct selectors automatically

### Limitation

* Cannot specify `nodePort`

---

## Alternative: Create NodePort with Specific Port

```bash
kubectl create service nodeport nginx --tcp=80:80 --node-port=30080 --dry-run=client -o yaml
```

### Advantages

* Allows specifying `nodePort`

### Limitations

* Uses default (often incorrect) selectors

---

# Best Practice for Services

Recommended workflow:

1. Use `kubectl expose` to generate YAML (correct selectors)
2. Edit YAML to add `nodePort` if needed
3. Apply configuration:

```bash
kubectl apply -f service.yaml
```

---

# Output Formatting

Kubectl supports multiple output formats using `-o` flag:

```bash
kubectl [command] [TYPE] [NAME] -o <format>
```

## Common Formats

### JSON

```bash
-o json
```

* Machine-readable
* Useful for scripting

---

### YAML

```bash
-o yaml
```

* Human-readable
* Best for editing configurations

---

### Wide

```bash
-o wide
```

* Shows additional details (IP, node, etc.)

---

### Name Only

```bash
-o name
```

* Outputs only resource names
* Useful in scripts and pipelines

---

# Kubectl Explain Command

Provides built-in documentation for Kubernetes resources.

---

## List API Resources - List all resource types (Pods, Services, Deployments, etc.) that the cluster API supports, including whether they are namespaced.

```bash
kubectl api-resources
```

---

## Explain a Resource - Show documentation for the Pod resource, describing its purpose and top-level fields:

```bash
kubectl explain pods
```

---

## Explain Specific Field - Dive into the Pod’s spec section, explaining how to configure containers, volumes, and other Pod specifications.

```bash
kubectl explain pods.spec
```

---

## Full Recursive Documentation - Show a full, detailed explanation of all Pod fields and subfields recursively, giving complete documentation for the resource structure.

```bash
kubectl explain pods --recursive
```

---

# Key Takeaways

* Use **imperative commands** for speed and prototyping
* Use **declarative YAML** for production and version control
* Combine `--dry-run=client` and `-o yaml` to generate templates quickly
* Always verify selectors when creating Services imperatively
* Use `kubectl explain` as a built-in reference tool
