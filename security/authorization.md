# Authorization in Kubernetes

---

## What is Authorization?

Authorization in Kubernetes is the process of **determining what an authenticated user, service, or component is allowed to do**.

After a request is authenticated (identity verified), the **kube-apiserver evaluates permissions** to decide:

"Is this entity allowed to perform this action on this resource?"

---

## Why Do We Need Authorization?

Without authorization:
- Any authenticated user could perform **any action**
- Critical resources (Pods, Secrets, Nodes) could be modified or deleted
- Cluster security would be compromised

Authorization ensures:
- Controlled access to resources  
- Enforcement of least privilege  
- Separation of responsibilities (e.g., dev vs admin)  

---

## Authorization Flow

1. Request reaches kube-apiserver  
2. Authentication verifies identity  
3. Authorization evaluates permissions  
4. Request is:
   - Allowed → executed  
   - Denied → rejected  

---

# Authorization Mechanisms

Kubernetes supports multiple authorization modes:

- Node Authorizer  
- ABAC (Attribute-Based Access Control)  
- RBAC (Role-Based Access Control)  
- Webhook  
- AlwaysAllow (default in some setups, not recommended)  
- AlwaysDeny  

---

# 1. Node Authorizer

## What is Node Authorizer?

The **Node Authorizer** is a special-purpose authorization mode designed for **kubelets (nodes)**.

It ensures that each node:
- Can only access resources **related to itself**
- Cannot access or modify resources belonging to other nodes

---

## Why is it Needed?

Kubelets constantly interact with the API server to:

- Read Pods assigned to them  
- Fetch Secrets and ConfigMaps for those Pods  
- Report node status  
- Update Pod status  

Without restrictions, a compromised node could:
- Access Secrets of other Pods  
- Interfere with workloads on other nodes  

---

## How Node Authorizer Works

- Each kubelet authenticates as a user:
```system:node:<node-name>```

* The Node Authorizer checks:

  * Is this request coming from a valid node?
  * Is the node requesting resources assigned to it?

---

## What Nodes Can Do

Allowed:

* Read Pods scheduled on that node
* Read Secrets/ConfigMaps used by those Pods
* Update their own Node status
* Update Pod status

---

## What Nodes Cannot Do

Denied:

* Access Pods on other nodes
* Access unrelated Secrets
* Modify cluster-wide resources

---

## Node Restriction Admission Controller

Often used together with Node Authorizer.

It:

* Prevents kubelets from modifying labels or resources they shouldn’t
* Adds an extra layer of security

---

# 2. ABAC (Attribute-Based Access Control)

## What is ABAC?

ABAC grants permissions based on **attributes**:

* User
* Group
* Resource
* Action

---

## How It Works

* Policies are defined in a JSON file
* Passed to kube-apiserver using:

```--authorization-policy-file=<file>```

---

## Example Concept

A policy might allow:

* User "dev-user"
* To perform "get", "create", "delete"
* On Pods

---

## Limitations

* Policies are static
* No grouping or abstraction
* Changes require editing files manually
* Not scalable

ABAC is **largely deprecated** in favor of RBAC

---

# 3. RBAC (Role-Based Access Control) - Standard Approach

## What is RBAC?

RBAC defines permissions using **roles**, then assigns those roles to users or groups.

---

## How It Works

1. Create a Role (set of permissions)
2. Bind the Role to users/groups

---

## Example Concept

* Role: "developer"

  * Can: get, list, create Pods

* Bind Role to:

  * dev-team group

---

## Benefits

* Easier to manage
* Scalable
* Reusable roles
* Fine-grained control

This is the **recommended approach in Kubernetes**

---

# 4. Webhook Authorization

## What is Webhook Authorization?

Webhook allows Kubernetes to **delegate authorization decisions to an external system**.

---

## How It Works

1. Request reaches kube-apiserver
2. API server sends request details to an external service (via HTTP)
3. External service evaluates the request
4. Returns:

   * Allow
   * Deny

---

## Example Use Case

Using **Open Policy Agent (OPA)**:

* Define custom policies (e.g., "no containers with root user")
* Enforce organization-wide rules

---

## Request Sent to Webhook Includes

* User identity
* Resource
* Action (verb)
* Namespace

---

## Response from Webhook

```
{
  "allowed": true
}
```

or

```
{
  "allowed": false
}
```

---

## When to Use

* Advanced security requirements
* Custom policies beyond RBAC
* Integration with enterprise systems

---

# 5. AlwaysAllow / AlwaysDeny

## AlwaysAllow

* Allows ALL requests
* No authorization checks

## AlwaysDeny

* Denies ALL requests

---

## Use Cases

* Testing only
* Never use in production

---

# Multiple Authorization Modes

You can configure multiple modes:

```--authorization-mode=Node,RBAC,Webhook```

---

## Important Behavior (Order Matters!)

* Modes are evaluated **in order**
* If one authorizer:

  * **Approves** → request is allowed immediately
  * **Denies** → move to next authorizer
* If all deny → request is rejected

---

## Example Flow

```Request → Node → RBAC → Webhook```

* Node: Deny
* RBAC: Allow → STOP → request allowed

---

# Summary

* Authorization defines **what actions are allowed**
* It follows authentication
* Multiple mechanisms exist:

  * Node Authorizer (for kubelets)
  * ABAC (legacy)
  * RBAC (recommended)
  * Webhook (external policies)
* Order of evaluation matters when combining modes
* RBAC + Node Authorizer is the most common production setup