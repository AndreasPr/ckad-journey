# Creating Roles in Kubernetes RBAC

---

## What is a Role?

A **Role** in Kubernetes RBAC defines a set of **permissions (rules)** within a specific namespace.

It answers: "What actions are allowed on which resources?"

A Role does NOT assign permissions to users directly.  
Instead, it defines permissions that can later be **granted via RoleBindings**.

---

# Step 1: Create a Role Definition

You define a Role using a YAML file.

## Example: developer-role.yaml

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: developer
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["list", "get", "create", "update", "delete"]

- apiGroups: [""]
  resources: ["configmaps"]
  verbs: ["create"]
```

---

## Explanation

### apiGroups

* Specifies which API group the resource belongs to
* `""` (empty string) refers to the **Core API group**

  * Examples: pods, services, configmaps

---

### resources

* Defines which Kubernetes objects the rule applies to
* Examples:

  * pods
  * configmaps
  * deployments (in `apps` group)

---

### verbs

Defines allowed actions:

* get → retrieve a resource
* list → list multiple resources
* create → create new resources
* update → modify existing resources
* delete → remove resources

---

# Step 2: Create the Role

```kubectl create -f developer-role.yaml```

---

# Step 3: Bind the Role to a User (RoleBinding)

Roles do nothing until they are **assigned to a subject**.

This is done using a **RoleBinding**.

---

## Example: devuser-developer-binding.yaml

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: devuser-developer-binding

subjects:
- kind: User
  name: dev-user
  apiGroup: rbac.authorization.k8s.io

roleRef:
  kind: Role
  name: developer
  apiGroup: rbac.authorization.k8s.io
```

---

## Explanation

### subjects

Defines WHO gets access:

* kind:

  * User
  * Group
  * ServiceAccount

* name:

  * The actual identity (e.g., `dev-user`)

---

### roleRef

Defines WHICH role is assigned:

* kind: Role
* name: developer
* apiGroup: rbac.authorization.k8s.io

---

# Step 4: Create the RoleBinding

```kubectl create -f devuser-developer-binding.yaml```

---

# Viewing and Inspecting RBAC Objects

---

### List Roles

```kubectl get roles```

---

### List RoleBindings

```kubectl get rolebindings```

---

### Describe a Role

```kubectl describe role <role-name>```

---

### Describe a RoleBinding

```kubectl describe rolebinding <rolebinding-name>```

---

# Checking Access (Authorization Testing)

Kubernetes provides a built-in way to verify permissions.

---

## Check Your Own Permissions

```kubectl auth can-i create deployments```

```kubectl auth can-i delete nodes```

---

## Check Another User's Permissions (Impersonation)

```kubectl auth can-i create deployments --as dev-user```

---

## Check Permissions in a Specific Namespace

```kubectl auth can-i create pods --as dev-user --namespace test```

---

# Restricting Access to Specific Resources

You can limit access to **specific resource instances** using `resourceNames`.

---

## Example

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: developer
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "create", "delete"]
  resourceNames: ["blue", "orange"]
```

---

## What This Means

The user can:

* Access only Pods named:

  * blue
  * orange

The user CANNOT:

* Access any other Pods

---

# Important Notes

* Roles are **namespace-scoped**
* For cluster-wide access, use **ClusterRole** and **ClusterRoleBinding**
* RBAC does not create users; it only assigns permissions
* Always follow the **principle of least privilege**

---

# Summary

* A Role defines permissions (resources + verbs)
* A RoleBinding assigns those permissions to users/groups
* Use `kubectl auth can-i` to verify access
* Use `resourceNames` for fine-grained control
* RBAC is the standard and recommended authorization method in Kubernetes



# Scenarios
## Scenario 1
Inspect the environment and identify the authorization modes configured on the cluster.
Check the `kube-apiserver` settings.
## Solution
Use the command `kubectl describe pod kube-apiserver-controlplane -n kube-system` and look for `--authorization-mode`.

or
`cat /etc/kubernetes/manifests/kube-apiserver.yaml`

or
`ps -aux | grep authorization`

## Scenario 2
How many roles exist in all namespaces together?
## Solution
Run the command: `kubectl get roles --all-namespaces` or `kubectl get roles -A`

In order to get the exact number immediately: `kubectl get roles -A --no-headers | wc -l`

## Scenario 3
What are the resources the kube-proxy role in the kube-system namespace is given access to?
## Solution
Run the command: `kubectl describe role kube-proxy -n kube-system`

## Scenario 4
Which account is the `kube-proxy` role assigned to?
## Solution
Run the command: `kubectl describe rolebinding kube-proxy -n kube-system`

## Scenario 5
A user `dev-user` is created. User's details have been added to the `kubeconfig` file. Inspect the permissions granted to the user. Check if the user can list pods in the `default` namespace.
Use the `--as dev-user` option with `kubectl` to run commands as the `dev-user`

## Solution
Run the command: `kubectl get pods --as dev-user`

## Scenario 6
Create the necessary roles and role bindings required for the `dev-user` to create, list and delete pods in the `default` namespace.

Use the given spec:
* Role: developer
* Role Resources: pods
* Role Actions: list
* Role Actions: create
* Role Actions: delete
* RoleBinding: dev-user-binding
* RoleBinding: Bound to dev-user

## Solution
To create a Role:- `kubectl create role developer --namespace=default --verb=list,create,delete --resource=pods`

To create a RoleBinding:- `kubectl create rolebinding dev-user-binding --namespace=default --role=developer --user=dev-user`

OR

Solution manifest file to create a role and rolebinding in the default namespace:
```
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: default
  name: developer
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["list", "create","delete"]
```

---

```
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: dev-user-binding
subjects:
- kind: User
  name: dev-user
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: developer
  apiGroup: rbac.authorization.k8s.io
```

## Scenario 7
A set of new roles and role-bindings are created in the blue namespace for the dev-user. However, the `dev-user` is unable to get details of the `dark-blue-app` pod in the blue namespace. Investigate and fix the issue.


We have created the required roles and rolebindings, but something seems to be wrong.

## Solution
Run the command: `kubectl edit role developer -n blue` and correct the `resourceNames` field. You don't have to delete the role.


## Scenario 8
Add a new rule in the existing role `developer` to grant the `dev-user` permissions to create deployments in the `blue` namespace.
Remember to add api group `"apps"`.
## Solution
Edit the `developer` role in the `blue` namespace to add a new rule under the `rules` section.

Append the below rule to the end of the file

```kubectl edit role developer -n blue```

```
- apiGroups:
  - apps
  resources:
  - deployments
  verbs:
  - create
```
So it looks like this:
```
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: developer
  namespace: blue
rules:
- apiGroups:
  - apps
  resourceNames:
  - dark-blue-app
  resources:
  - pods
  verbs:
  - get
  - watch
  - create
  - delete
- apiGroups:
  - apps
  resources:
  - deployments
  verbs:
  - create
```