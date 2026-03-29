# Kubernetes Deployments & Rollouts

## What is a Rollout?

When you create or update a Deployment, Kubernetes performs a **rollout**.

A rollout:
- Creates a new **ReplicaSet**
- Gradually updates Pods
- Tracks changes as **revisions**

## Why Rollouts Matter

- Enable **version tracking**
- Allow **safe updates**
- Support **rollback to previous versions**

Each update = new **Deployment revision**


## Rollout Commands

### Check rollout status
```kubectl rollout status deployment/myapp-deployment```

Shows:
* Progress of the deployment
* Whether update is complete or still running


## View rollout history
```kubectl rollout history deployment/myapp-deployment```

Shows:
* All revisions of the Deployment
* Useful for debugging and rollback




## Deployment Strategies
### 1. Recreate Strategy

Behavior:
* Terminates all existing Pods
* Then creates new Pods

Drawback:
* Causes downtime

### 2. Rolling Update (Default)

Behavior:
* Gradually replaces old Pods with new ones
* Ensures some Pods are always running

Benefits:
* Zero downtime
* Smooth transition



## Updating a Deployment
### Example: Update Image Version

Update container image from:

`image: nginx`

To:

`image: nginx:1.7.1`

**Option 1: Apply YAML (Recommended)**

```kubectl apply -f deployment-definition.yaml```

Best practice:
* Keeps configuration in sync with your YAML file
* Ideal for version control (GitOps)

**Option 2: Imperative Command**

```kubectl set image deployment/myapp-deployment nginx=nginx:1.7.1```

Notes:
* Updates image directly in the cluster
* DOES NOT update your local YAML file

Risk:
* Configuration drift between cluster and code



## Rollback a Deployment
```kubectl rollout undo deployment/myapp-deployment```

What happens:
* Kubernetes reverts to the previous revision
* Old ReplicaSet becomes active again
* New Pods are terminated

### Rollback to Specific Revision
```kubectl rollout undo deployment/myapp-deployment --to-revision=2```

Useful when multiple updates exist






## Behind the Scenes

Every update:

1. Creates a new ReplicaSet
2. Scales up new Pods
3. Scales down old Pods
4. Stores revision history


## Commands Summary
### Create Deployment
```kubectl create -f deployment-definition.yaml```

### Get Deployments
```kubectl get deployments```

### Update Deployment
```kubectl apply -f deployment-definition.yaml```

```kubectl set image deployment/myapp-deployment nginx=nginx:1.7.1```

### Check Status
```kubectl rollout status deployment/myapp-deployment```

```kubectl rollout history deployment/myapp-deployment```

#### Check the status of each revision individually by using the `--revision` flag
```kubectl rollout history deployment nginx --revision=1```


### Rollback
```kubectl rollout undo deployment/myapp-deployment```

#### Rollback to specific revision
```kubectl rollout undo deployment nginx --to-revision=1```


## Key Takeaways
* Rollout = Deployment update process
* Each update creates a new ReplicaSet (revision)
* Rolling Update = default, zero-downtime strategy
* `apply` > `set image` (avoids config drift)
* Rollbacks are fast and built-in

## Pro Tip

Always use:

* `kubectl rollout status` after updates
* Version-controlled YAML files

This ensures safe and predictable deployments


## Interesting tip
The `--record` flag in Kubernetes was used to automatically record the command that caused a change in the resource's annotations, which would then appear in the CHANGE-CAUSE column of the kubectl rollout history output. Instead of using the removed `--record` flag, the professional standard today is to manually add the `kubernetes.io/change-cause` annotation immediately after making a change. This provides a clear audit trail for debugging and incident response.

#### Here is the modern approach:
1. Perform the desired action (e.g., set a new image):

```kubectl set image deployment/api-server api-server=my-repo/api:v1.10.2```

2. Manually annotate the deployment with a descriptive reason:

```kubectl annotate deployment/api-server kubernetes.io/change-cause="Rolled back to v1.10.2 due to high latency in v1.10.3"```

3. View the history which will now show the cause:

```kubectl rollout history deployment/api-server```

The output will include the `CHANGE-CAUSE` message provided in the annotation.