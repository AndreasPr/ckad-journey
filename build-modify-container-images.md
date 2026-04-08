# Building a Docker Image (Flask App)

This example demonstrates how to build a Docker image for a simple Python Flask application.

---

## Requirements

The container setup includes:

1. Base OS: Ubuntu
2. Update package repository
3. Install system dependencies
4. Install Python dependencies
5. Copy application source code
6. Run the Flask web server

---

## Dockerfile

```dockerfile
FROM ubuntu

# Update package repository
RUN apt-get update

# Install Python and pip
RUN apt-get install -y python3 python3-pip

# Install Python dependencies
RUN pip3 install flask
RUN pip3 install flask-mysql

# Copy application code
COPY . /opt/source-code

# Set working directory (recommended)
WORKDIR /opt/source-code

# Set environment variable and run Flask app
ENV FLASK_APP=app.py

ENTRYPOINT ["flask", "run"]
```

---

# Build the Image

```bash
docker build -t andreas/my-app .
```

---

# Push to Docker Hub

```bash
docker push andreas/my-app
```

---

# Basic Docker Commands

## Run a Container

```bash
docker run ubuntu
```

---

## List Running Containers

```bash
docker ps
```

---

## List All Containers (Including Stopped)

```bash
docker ps -a
```

---

# Important Concepts

## Why Containers Exit Immediately

By default:

* Docker does **not attach a terminal**
* If the main process exits → container stops

Example:

* Running `ubuntu` without a command → exits immediately

---

# ENTRYPOINT vs CMD

## ENTRYPOINT

* Defines the **main executable**
* Always runs when container starts

Example:

```dockerfile
ENTRYPOINT ["sleep"]
```

---

## CMD

* Provides **default arguments** to ENTRYPOINT
* Can be overridden at runtime

Example:

```dockerfile
CMD ["5"]
```

---

## ENTRYPOINT + CMD (Best Practice)

```dockerfile
ENTRYPOINT ["sleep"]
CMD ["5"]
```

Equivalent to:

```bash
sleep 5
```

---

## Override at Runtime

```bash
docker run ubuntu-sleeper 10
```

Runs:

```bash
sleep 10
```

---

# Kubernetes Override Behavior

When running containers in Kubernetes:

## Pod Definition Example

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper-pod
spec:
  containers:
    - name: ubuntu-sleeper
      image: ubuntu-sleeper
      command: ["sleep"]
      args: ["10"]
```

---

## Key Rules

* `command` → overrides **ENTRYPOINT**
* `args` → overrides **CMD**

---

## Mapping Summary

| Dockerfile | Kubernetes |
| ---------- | ---------- |
| ENTRYPOINT | command    |
| CMD        | args       |

---

## Example

### Dockerfile

```dockerfile
FROM ubuntu
ENTRYPOINT ["sleep"]
CMD ["5"]
```

---

### Kubernetes Pod

```yaml
command: ["sleep"]
args: ["10"]
```

---

### Result

```bash
sleep 10
```

---

# Key Takeaways

* Use `ENTRYPOINT` for the main command
* Use `CMD` for default arguments
* Kubernetes overrides:

  * `command` → ENTRYPOINT
  * `args` → CMD
* Always design containers to run a **single main process**
* Use `--dry-run` and YAML generation for better control in Kubernetes
