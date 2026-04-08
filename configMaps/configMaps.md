# ConfigMaps in Kubernetes

ConfigMaps are Kubernetes objects used to store non-sensitive configuration data as key-value pairs. They allow you to decouple configuration from application code, making your applications more portable and easier to manage across environments.

## Workflow

1. Create a ConfigMap
2. Inject the ConfigMap into a Pod

---

## Example: Using ConfigMap in a Pod

### ConfigMap Definition

*config-map.yaml*
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_COLOR: blue
  APP_MODE: prod
````

### Pod Definition

*pod-definition.yaml*

```yaml
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
        - configMapRef:
            name: app-config
```

### Explanation

* `envFrom` imports **all key-value pairs** from the ConfigMap as environment variables.
* `APP_COLOR` and `APP_MODE` become available inside the container automatically.
* This avoids defining each variable manually.

---

## Ways to Create ConfigMaps

There are **two main approaches**:

---

## 1. Imperative Approach

Used for quick creation directly from the command line.

### From Literal Values

```bash
kubectl create configmap <config-name> --from-literal=<key>=<value>
```

Example:

```bash
kubectl create configmap app-config \
  --from-literal=APP_COLOR=blue \
  --from-literal=APP_MODE=prod
```

Explanation:

* Each `--from-literal` adds one key-value pair
* Creates a ConfigMap instantly without YAML

---

### From File

```bash
kubectl create configmap <config-name> --from-file=<path-to-file>
```

Example:

```bash
kubectl create configmap app-config --from-file=app_config.properties
```

If `app_config.properties` contains:

```
APP_COLOR=blue
APP_MODE=prod
```

Then:

* The file name becomes the key
* The entire file content becomes the value

Use case:

* Large configurations
* Application config files (e.g., `.properties`, `.env`, `.json`)

---

## 2. Declarative Approach (Recommended)

Best for production environments because it is:

* Version-controlled
* Repeatable
* Easier to maintain

### Step 1: Define YAML

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_COLOR: blue
  APP_MODE: prod
```

### Step 2: Apply Configuration

```bash
kubectl apply -f config-map.yaml
```

Explanation:

* Creates or updates the ConfigMap
* Preferred over `create` for idempotency

---

## Methods to Inject ConfigMap into Pods

### 1. Using `envFrom` (Import All Variables)

```yaml
envFrom:
  - configMapRef:
      name: app-config
```

* Loads all keys as environment variables

---

### 2. Using `env` (Selective Injection)

```yaml
env:
  - name: APP_COLOR
    valueFrom:
      configMapKeyRef:
        name: app-config
        key: APP_COLOR
```

* Allows fine-grained control
* Only specific keys are injected

---

### 3. Mounting as Files (Advanced)

```yaml
volumeMounts:
  - name: config-volume
    mountPath: /etc/config

volumes:
  - name: config-volume
    configMap:
      name: app-config
```

* Each key becomes a file inside the directory
* Useful for applications expecting config files

---

## Commands

### List ConfigMaps

```bash
kubectl get configmaps
```

or

```bash
kubectl get cm
```

---

### Describe ConfigMap

```bash
kubectl describe configmap <config-name>
```

* Shows detailed information including stored data

---

## Best Practices

* Use ConfigMaps for **non-sensitive configuration only**
* Use **Secrets** for credentials and sensitive data
* Prefer the **declarative approach** in production
* Use **envFrom** for simplicity and **env** for control
* Use **volume mounts** when applications expect config files

---

## Summary

ConfigMaps provide a clean and scalable way to manage application configuration in Kubernetes. They separate configuration from code, improve reusability, and simplify environment-specific customization.

