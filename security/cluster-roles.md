# ClusterRole in Kubernetes

---

## What is a ClusterRole?

A **ClusterRole** is an RBAC object that defines a set of permissions **at the cluster level**.

It is similar to a **Role**, but with one key difference:

- A **Role** applies to a specific namespace  
- A **ClusterRole** applies to the **entire cluster**

---

## Resource Types in Kubernetes

Kubernetes resources are divided into two categories:

### 1. Namespaced Resources
These exist within a namespace.

Examples:
- Pods  
- Deployments  
- Services  
- ReplicaSets  
- Jobs  
- Roles  
- RoleBindings  

---

### 2. Cluster-Scoped Resources
These do NOT belong to any namespace.

Examples:
- Nodes  
- PersistentVolumes (PV)  
- Namespaces  
- ClusterRoles  
- ClusterRoleBindings  
- CertificateSigningRequests  

---

## How to List Them

```kubectl api-resources --namespaced=true```

```kubectl api-resources --namespaced=false```

---

# Why Do We Need ClusterRoles?

Some resources are **cluster-wide**, so they cannot be managed using Roles.

For example:

* Viewing all nodes in the cluster
* Managing PersistentVolumes
* Accessing all namespaces

To authorize users for these, we use:

* **ClusterRole**
* **ClusterRoleBinding**

---

# Step 1: Create a ClusterRole

## Example: cluster-admin-role.yaml

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cluster-administrator

rules:
- apiGroups: [""]
  resources: ["nodes"]
  verbs: ["list", "get", "create", "delete"]
```

---

## Explanation

* apiGroups: `""` → Core API group
* resources: `nodes` → cluster-scoped resource
* verbs:

  * list → view all nodes
  * get → view specific node
  * create/delete → manage nodes

---

## Create the ClusterRole

```kubectl create -f cluster-admin-role.yaml```

---

# Step 2: Bind the ClusterRole to a User

A ClusterRole must be linked using a **ClusterRoleBinding**.

---

## Example: cluster-admin-role-binding.yaml

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: cluster-admin-role-binding

subjects:
- kind: User
  name: cluster-admin
  apiGroup: rbac.authorization.k8s.io

roleRef:
  kind: ClusterRole
  name: cluster-administrator
  apiGroup: rbac.authorization.k8s.io
```

---

## Create the ClusterRoleBinding

```kubectl create -f cluster-admin-role-binding.yaml```

---

# Important Notes

---

## 1. ClusterRole for Namespaced Resources

You can also use a **ClusterRole** for namespaced resources.

Example:

* pods
* deployments

But in this case:

The user gets access to these resources **across ALL namespaces**

---

## 2. Default ClusterRoles

Kubernetes creates several ClusterRoles by default, such as:

* cluster-admin → full access to everything
* admin → full access within a namespace
* edit → modify resources but not RBAC
* view → read-only access

---

## 3. ClusterRole vs Role

| Feature    | Role             | ClusterRole           |
| ---------- | ---------------- | --------------------- |
| Scope      | Namespace        | Entire cluster        |
| Works with | RoleBinding      | ClusterRoleBinding    |
| Use case   | App-level access | Infra / global access |

---

## 4. Binding Scope Matters

* **RoleBinding + ClusterRole**
  → Grants access ONLY within a namespace

* **ClusterRoleBinding + ClusterRole**
  → Grants access across the entire cluster

---

# Example Use Cases

---

## Use ClusterRole when:

* You need access to:

  * Nodes
  * PersistentVolumes
  * Namespaces

* You want:

  * Read access to all pods across all namespaces
  * Centralized admin permissions

---


# Commands
`kubectl get clusterroles`

`kubectl describe clusterrole {name-cluster-role} `

`kubectl describe clusterrolebinding {name-cluster-role-binding}`

# Summary

* ClusterRole defines permissions at the **cluster level**
* Used for both:

  * Cluster-scoped resources
  * Cross-namespace access
* Requires ClusterRoleBinding to assign permissions
* Essential for admin-level and infrastructure operations




# Scenarios
## Scenario 1
How many ClusterRoles do you see defined in the cluster?

## Solution
Run the command: `kubectl get clusterroles --no-headers  | wc -l` or `kubectl get clusterroles --no-headers  -o json | jq '.items | length'`


## Scenario 2
How many ClusterRoleBindings exist on the cluster?

## Solution
Run the command: `kubectl get clusterrolebindings --no-headers  | wc -l` or `kubectl get clusterrolebindings --no-headers  -o json | jq '.items | length'`

## Scenario 3
What namespace is the cluster-admin clusterrole part of?

## Solution
ClusterRole is a non-namespaced resource. You can check via the `kubectl api-resources --namespaced=false` command. So the correct answer would be Cluster Roles are cluster wide and not part of any namespace.

## Scenario 4
What user/groups are the cluster-admin role bound to?
The ClusterRoleBinding for the role is with the same name.


## Solution
Run the command: `kubectl describe clusterrolebinding cluster-admin`

## Scenario 5
What level of permission does the cluster-admin role grant?

Inspect the cluster-admin role's privileges.

## Solution
Run the command: `kubectl describe clusterrole cluster-admin`


## Scenario 6
A new user michelle joined the team. She will be focusing on the nodes in the cluster. Create the required ClusterRoles and ClusterRoleBindings so she gets access to the nodes.

## Solution
Solution manifest file to create a clusterrole and clusterrolebinding for `michelle` user:

```
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: node-admin
rules:
- apiGroups: [""]
  resources: ["nodes"]
  verbs: ["get", "watch", "list", "create", "delete"]

---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: michelle-binding
subjects:
- kind: User
  name: michelle
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: node-admin
  apiGroup: rbac.authorization.k8s.io
```
After save into a file, run the command `kubectl create -f <file-name>.yaml` to create a resources from definition file.

## Scenario 7
`michelle`'s responsibilities are growing and now she will be responsible for storage as well. Create the required `ClusterRoles` and `ClusterRoleBindings` to allow her access to Storage.

Get the API groups and resource names from command kubectl api-resources. Use the given spec:

- ClusterRole: storage-admin
- Resource: persistentvolumes
- Resource: storageclasses
- ClusterRoleBinding: michelle-storage-admin
- ClusterRoleBinding Subject: michelle
- ClusterRoleBinding Role: storage-admin


## Solution

Solution manifest file to create a clusterrole and clusterrolebinding for michelle user:

```
---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: storage-admin
rules:
- apiGroups: [""]
  resources: ["persistentvolumes"]
  verbs: ["get", "watch", "list", "create", "delete"]
- apiGroups: ["storage.k8s.io"]
  resources: ["storageclasses"]
  verbs: ["get", "watch", "list", "create", "delete"]

---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: michelle-storage-admin
subjects:
- kind: User
  name: michelle
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: storage-admin
  apiGroup: rbac.authorization.k8s.io
```

After save into a file, run the command kubectl create -f <file-name>.yaml to create a resources from definition file.