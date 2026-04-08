## Security Context in Kubernetes

A **Security Context** in Kubernetes defines **privilege and access control settings** for a Pod or a container.
It allows you to control **how containers run from a security perspective**, such as:

* Which **user ID** the container runs as
* Which **Linux capabilities** it has
* Whether it can run as **root**
* Filesystem permissions
* Privileged mode

The main goal is to **limit container privileges** and follow the **principle of least privilege**, improving cluster security.

---

# Container Security (Docker Examples)

### Run container as a specific user (non-root)

This runs an Ubuntu container as **user ID 1001** instead of root and keeps it alive for 1 hour:

```bash
docker run --user=1001 ubuntu sleep 3600
```

Why this matters:

* Containers run as **root by default**
* Running as **non-root reduces security risks**

---

### Add Linux capabilities

This runs a container with the **MAC_ADMIN capability**, allowing it to modify security policies such as SELinux/AppArmor.

```bash
docker run --cap-add MAC_ADMIN ubuntu
```

Linux capabilities allow containers to perform **specific privileged actions** without giving full root privileges.

---

# Pod vs Container Security Context

Security contexts can be defined at **two levels**:

| Level           | Scope                                    |
| --------------- | ---------------------------------------- |
| Pod Level       | Applies to **all containers in the Pod** |
| Container Level | Applies to **only that container**       |

### Important Rule

If both are defined:

**Container-level settings override Pod-level settings.**

---

# Security Context at the Pod Level

Example:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-pod
spec:
  securityContext:
    runAsUser: 1000
  containers:
    - name: ubuntu
      image: ubuntu
      command: ["sleep", "3600"]
```

### What happens here

* The **Pod runs as user 1000**
* All containers inherit this user unless overridden

---

# Security Context at the Container Level

Example:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-pod
spec:
  containers:
    - name: ubuntu
      image: ubuntu
      command: ["sleep", "3600"]
      securityContext:
        runAsUser: 1000
```

Here:

* Only the **ubuntu container** runs as user **1000**

---

# Linux Capabilities in Kubernetes

Capabilities allow containers to perform **specific privileged operations**.

Important:

⚠️ **Capabilities can only be configured at the container level**, not the Pod level.

Example:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-pod
spec:
  containers:
    - name: ubuntu
      image: ubuntu
      command: ["sleep", "3600"]
      securityContext:
        runAsUser: 1000
        capabilities:
          add: ["MAC_ADMIN"]
```

### What this does

* Container runs as **user 1000**
* Adds the **MAC_ADMIN capability**
* Allows modification of security policies

---

# Common SecurityContext Fields

| Field                      | Description                            |
| -------------------------- | -------------------------------------- |
| `runAsUser`                | User ID used to run the container      |
| `runAsGroup`               | Group ID for container process         |
| `runAsNonRoot`             | Ensures container cannot run as root   |
| `privileged`               | Gives full host privileges (dangerous) |
| `allowPrivilegeEscalation` | Prevents gaining more privileges       |
| `readOnlyRootFilesystem`   | Makes container filesystem read-only   |
| `capabilities`             | Adds or drops Linux capabilities       |

Example:

```yaml
securityContext:
  runAsUser: 1000
  runAsNonRoot: true
  allowPrivilegeEscalation: false
  readOnlyRootFilesystem: true
```

---

# Security Best Practices

1. **Run containers as non-root**
2. **Drop unnecessary capabilities**
3. Use **read-only filesystems**
4. Disable **privilege escalation**
5. Avoid **privileged containers**

Example hardened container:

```yaml
securityContext:
  runAsNonRoot: true
  runAsUser: 1000
  allowPrivilegeEscalation: false
  readOnlyRootFilesystem: true
```

---

# Summary

A **Security Context** controls **how containers run securely in Kubernetes**.

It helps enforce:

* Non-root execution
* Capability restrictions
* Filesystem protections
* Least privilege access

Security contexts can be applied at:

* **Pod level → affects all containers**
* **Container level → overrides pod settings**

And **Linux capabilities must always be defined at the container level**.