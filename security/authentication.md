# Authentication in Kubernetes

---

## What is Authentication?

Authentication in Kubernetes is the process of **verifying the identity of a user, service, or system** that is trying to access the cluster.

Before any request is processed, the **kube-apiserver** must determine:

"Who is making this request?"

Only after successful authentication does Kubernetes move to the next step: **authorization** (what the user is allowed to do).

---

## Key Concept

Kubernetes **does NOT manage user accounts natively**.

Instead, it relies on **external identity sources** such as:
- Files (tokens, certificates)
- Identity providers (LDAP, OIDC, etc.)
- Cloud IAM systems

---

## Exception: Service Accounts

Kubernetes **does manage service accounts internally**.

- Used by applications (Pods) to interact with the API server  
- Automatically created and managed within namespaces  
- Provide identity for workloads running inside the cluster  

---

## How Authentication Works

All access to the cluster goes through the `kube-apiserver`, including:

- `kubectl` commands  
- Direct API calls (e.g., using `curl`)  

Example:
```kubectl get pods```

or

```curl https://<kube-apiserver-ip>:6443/api/v1/pods```

---

### Authentication Flow

1. Client sends request to `kube-apiserver`
2. API server verifies identity using configured authentication method
3. If valid → request proceeds to authorization
4. If invalid → request is rejected

---

## Supported Authentication Mechanisms

---

### 1. Static Token File (NOT Recommended)

* A file containing predefined user-token mappings
* Passed to `kube-apiserver` using `--token-auth-file`

Example token file:

```
<token>,<username>,<userid>,<groups>
```

Example usage:

```
curl -k https://<master-node-ip>:6443/api/v1/pods \
  -H "Authorization: Bearer <token>"
```

---

#### Limitations

* Tokens are static (no expiration)
* Difficult to rotate and manage
* NOT secure for production environments

---

### 2. Certificates (X.509)

* Most common and secure method
* Users are authenticated using **client certificates**
* Certificates are issued and signed by a trusted **Certificate Authority (CA)**

Used by:

* Administrators (`kubectl`)
* Internal components (kubelet, scheduler, etc.)

---

### 3. Identity Services (External Providers)

Kubernetes can integrate with external identity systems:

* LDAP
* OIDC (OpenID Connect)
* Cloud providers (AWS IAM, Azure AD, Google IAM)

Benefits:

* Centralized authentication
* Easier user management
* Supports enterprise security policies

---

## Important Notes

---

### `kube-apiserver` is the Gatekeeper

* All requests MUST go through the API server
* Authentication is always enforced before any operation

---

### Volume Mount Consideration (kubeadm)

If using static token files:

* The file must be available to the `kube-apiserver`
* Typically mounted as a volume in kubeadm setups

---

### Always Combine with Authorization

Authentication only answers:

"Who are you?"

You must also configure authorization (e.g., `RBAC`) to answer:

"What are you allowed to do?"

---

## Best Practices

* Avoid static token files in production
* Use certificates or external identity providers
* Implement RBAC for access control
* Regularly rotate credentials
* Use short-lived tokens where possible

---

## Summary

* Authentication verifies identity before allowing access
* Kubernetes relies on external systems for user management
* Service accounts are managed internally for workloads
* `kube-apiserver` enforces authentication for all requests
* Secure methods include certificates and identity providers

