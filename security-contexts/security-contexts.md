# Container Security
### Run a container from the Ubuntu image as user ID 1001 instead of root, and keeps it alive for 1 hour. This is used to enforce non-root execution for better security:
```docker run --user=1001 ubuntu sleep 3600```

### Run a container with the additional Linux capability MAC_ADMIN, allowing it to modify security policies (e.g., SELinux/AppArmor). This grants elevated privileges beyond the default container permissions:
```docker run -cap-add MAC_ADMIN ubuntu```

* If you choose to configure the security settings at a pod level, the settings will carry over to all containers within the pod.
* If you configure it at both the pod and the container, the settings on the container will override the settings on the pod.

## Configure Security context at the POD level:
```
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

## Configure Security context at the CONTAINER level:
```
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

**Note: Capabilities are ONLY supported at the container level and NOT at the pod level**

Therefore:

```
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
* Note: Specify a list of capabilities to add to the pod