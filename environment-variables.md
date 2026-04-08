# Environment Variables in Kubernetes

Environment variables are a common way to pass configuration data into containers. In Kubernetes, they allow you to decouple configuration from application code, making deployments more flexible and portable.

Kubernetes supports multiple ways to define environment variables inside a Pod:

## Types of Environment Variable Sources

### 1. Plain Key-Value

This is the simplest approach, where values are directly defined in the Pod specification.

Use this method for:
- Non-sensitive configuration
- Simple, static values

Example:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp
spec:
  containers:
  - name: simple-webapp
    image: simple-webapp
    ports:
      - containerPort: 8080
    env:
      - name: APP_COLOR
        value: red
````

Equivalent Docker command:

```bash
docker run -e APP_COLOR=red simple-webapp
```

---

### 2. ConfigMap

A ConfigMap is used to store non-sensitive configuration data separately from the Pod definition. This improves reusability and maintainability.

Benefits:

* Centralized configuration
* Reusable across multiple Pods
* Easy updates without modifying Pod YAML

Example ConfigMap:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_COLOR: blue
```

Using ConfigMap in a Pod:

```yaml
env:
  - name: APP_COLOR
    valueFrom:
      configMapKeyRef:
        name: app-config
        key: APP_COLOR
```

---

### 3. Secrets

Secrets are used to store sensitive data such as passwords, API keys, or tokens. They are similar to ConfigMaps but are intended for confidential information.

Key characteristics:

* Values are base64-encoded (not encrypted by default)
* Can be mounted as environment variables or files
* Better suited for sensitive data

Example Secret:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
data:
  APP_COLOR: cmVk  # base64 for "red"
```

Using Secret in a Pod:

```yaml
env:
  - name: APP_COLOR
    valueFrom:
      secretKeyRef:
        name: app-secret
        key: APP_COLOR
```

---

## Key Differences

| Feature     | Plain Key-Value | ConfigMap         | Secret            |
| ----------- | --------------- | ----------------- | ----------------- |
| Use Case    | Simple config   | Non-sensitive     | Sensitive data    |
| Storage     | Pod YAML        | Kubernetes object | Kubernetes object |
| Security    | None            | None              | Base64 encoded    |
| Reusability | Low             | High              | High              |

---

## Best Practices

* Use **ConfigMaps** for configuration that may change across environments (dev, staging, prod).
* Use **Secrets** for sensitive data (credentials, tokens).
* Avoid hardcoding values directly in Pod definitions unless they are truly static.
* Consider mounting ConfigMaps/Secrets as files for complex configurations instead of environment variables.

---

## Summary

Environment variables in Kubernetes provide a flexible way to configure applications without modifying code. By leveraging ConfigMaps and Secrets, you can build scalable, secure, and maintainable deployments.
