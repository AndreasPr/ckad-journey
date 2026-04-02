# Kubernetes Security Primitives

---

## What are Security Primitives?

Security primitives are the **fundamental building blocks used to secure a Kubernetes environment**.  
They define how access is controlled, how identities are verified, and how communication is protected.

At a high level, Kubernetes security focuses on:
- Securing the **infrastructure (hosts and nodes)**
- Securing the **control plane (API server and components)**
- Controlling **who can access the cluster**
- Controlling **what actions they can perform**
- Securing **communication between components**

---

# Securing the Hosts

Before securing Kubernetes itself, the underlying hosts (nodes) must be hardened.

## Best Practices

- Disable direct **root access**
- Disable **password-based authentication**
- Allow only **SSH key-based authentication**
- Keep the OS and packages updated
- Restrict network access using firewalls and security groups

These steps reduce the attack surface and prevent unauthorized access to the nodes.

---

# Securing the Kubernetes Cluster

The **kube-apiserver** is the central component of Kubernetes.

All operations—whether from users, tools, or internal components—go through the API server.

Because of this, security revolves around two fundamental questions:

---

## 1. Who can access the cluster? (Authentication)

Authentication verifies the identity of a user or system.

Kubernetes supports multiple authentication mechanisms:

### Static Token Files
- Predefined tokens stored in a file
- Simple but not recommended for production
- Hard to manage and rotate

---

### Certificates (X.509)
- Most common and secure method
- Each user/component has a certificate signed by a trusted Certificate Authority (CA)
- Used for both users and internal components

---

### External Authentication Providers
- Integrates with existing identity systems
- Examples:
  - LDAP
  - OIDC (e.g., Google, Azure AD)
- Enables centralized identity management

---

### Service Accounts
- Used by applications and processes inside the cluster
- Provide identity for Pods to interact with the API server
- Managed automatically by Kubernetes

---

## 2. What can they do? (Authorization)

Authorization determines what actions an authenticated entity is allowed to perform.

---

### RBAC (Role-Based Access Control)
- Most widely used method
- Permissions are defined using:
  - Roles / ClusterRoles
  - RoleBindings / ClusterRoleBindings
- Granular and flexible

---

### ABAC (Attribute-Based Access Control)
- Uses policies based on attributes (user, resource, action)
- Less flexible and harder to manage than RBAC
- Largely deprecated in favor of RBAC

---

### Node Authorization
- Special-purpose authorization for kubelets
- Ensures nodes can only access resources related to themselves

---

### Webhook Authorization
- Delegates authorization decisions to an external service
- Useful for custom or enterprise security requirements

---

# Securing Communication (TLS)

All communication inside a Kubernetes cluster is secured using **TLS (Transport Layer Security)**.

This ensures:
- Encryption of data in transit
- Authentication between components
- Protection against man-in-the-middle attacks

---

## Key Communication Channels

TLS is used between:

- kube-apiserver ↔ etcd  
- kube-apiserver ↔ kubelet  
- kube-apiserver ↔ kube-controller-manager  
- kube-apiserver ↔ kube-scheduler  
- kube-apiserver ↔ kube-proxy  

---

## Why TLS is Critical

- Prevents unauthorized access to sensitive data (e.g., secrets in etcd)
- Ensures only trusted components can communicate
- Maintains integrity and confidentiality of cluster operations

---

# Summary

Security in Kubernetes is built on a few core principles:

- Secure the **underlying hosts**
- Control **who can access the cluster** (Authentication)
- Control **what actions are allowed** (Authorization)
- Protect **all communication with TLS**

These primitives form the foundation for building a secure and production-ready Kubernetes environment.