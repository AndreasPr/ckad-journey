# Network Policies in Kubernetes

## What is a Network Policy?

A **NetworkPolicy** is a Kubernetes resource used to control **how Pods communicate with each other and with external networks**.

It acts like a **firewall for Pods**


## Why Use Network Policies?

- Restrict access between Pods  
- Improve security (zero-trust model)  
- Control ingress (incoming) and egress (outgoing) traffic  


# How Network Policies Work

By default:
- All Pods can communicate with each other (**allow all**)

Once a NetworkPolicy is applied:
- Traffic becomes **restricted**
- Only explicitly allowed traffic is permitted


## Example: Allow API → DB on Port 3306

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
    name: db-policy
spec:
    podSelector:
        matchLabels:
            role: db
    policyTypes:
    - Ingress
    ingress:
    - from:
        - podSelector:
            matchLabels:
                name: api-pod
        ports:
        - protocol: TCP
          port: 3306
```





### What Does This Do?

Applies to:
* Pods with label: `role: db`

Allows:
* Only Pods with label: `name: api-pod`

Only on:
* TCP port 3306

Blocks:
* ALL other traffic to DB Pods


## Important: Namespace Scope

Problem:

`podSelector` alone applies to same namespace only

But if combined improperly, it might allow broader access




### Create the network policy:
```kubectl create -f policy-definition.yaml```

* Network policies are enforced by the network solution implemented on Kubernetes cluster.
* Not all network solutions support network policies.
* Solutions that support Network Policies:
    - Kube-router
    - Calico
    - Romana
* Solutions that DO NOT support Network Policies:
    - Flannel





Our goal is to protect the database pod so that it does not allow access from any other pod except the API pod and only on port 3306:
```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
    name: db-policy
spec:
    podSelector:
        matchLabels:
            role: db
    policyTypes:
    - Ingress
    ingress:
    - from:
        - podSelector:
            matchLabels:
                name: api-pod
        ports:
        - protocol: TCP
          port: 3306
```

What if there are multiple API pods in the cluster with the same labels, but in different namespaces?
The current policy would allow any pod in any namespace with matching labels to reach the db pod. For that reason, we add a new selector called `namespaceSelector` property along with `podSelector` property. Therefore, we have:
```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
    name: db-policy
spec:
    podSelector:
        matchLabels:
            role: db
    policyTypes:
    - Ingress
    ingress:
    - from:
        - podSelector:
            matchLabels:
                name: api-pod
          namespaceSelector:
            matchLabels:
                kubernetes.io/metadata.name: prod
        ports:
        - protocol: TCP
          port: 3306
```

## Allow Traffic from External IPs
We could configure a network policy to allow traffic originating from certain IP addresses. For this reason, we use `ipBlock` field. Therefore:
```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
    name: db-policy
spec:
    podSelector:
        matchLabels:
            role: db
    policyTypes:
    - Ingress
    ingress:
    - from:
        - podSelector:
            matchLabels:
                name: api-pod
          namespaceSelector:
            matchLabels:
                kubernetes.io/metadata.name: prod
        - ipBlock:
            cidr: 192.168.5.10/32
        ports:
        - protocol: TCP
          port: 3306
```


### Explanation of the TWO Rules

```
ingress:
- from:
    - podSelector: {name: api-pod}
      namespaceSelector: {prod}
    - ipBlock: 192.168.5.10/32
```

These are OR conditions

* Rule 1:
Allow traffic from:
    * Pods with `name=api-pod`
    * In namespace `prod`

* Rule 2:
Allow traffic from:
    * External IP 192.168.5.10


Final Result:

* DB accepts traffic from:
    * API Pods (in prod namespace)
    * OR external IP





## Egress Rules (Outgoing Traffic)

By default:
* Pods can send traffic anywhere

With egress policy:
* You restrict outgoing traffic

## Example: DB → Backup Server
Instead of the backup server initiating a backup, say we have an agent on the db pod that pushes backup to the backup server. In that case, the traffic is originating from the db pod to an external backup server. So, we need egress rule defined. Therefore:
```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
    name: db-policy
spec:
    podSelector:
        matchLabels:
            role: db
    policyTypes:
    - Ingress
    - Egress
    ingress:
    - from:
        - podSelector:
            matchLabels:
                name: api-pod
        ports:
        - protocol: TCP
          port: 3306
    egress:
    - to: 
        - ipBlock:
            cidr: 192.168.5.10/32
        ports:
        - protocol: TCP
          port: 80
```

Allows DB Pods to:
* Send traffic to 192.168.5.10
* On port 80 only

## Important Notes
* Policies apply only to selected Pods
* If no policy exists -> all traffic allowed
* If policy exists -> only allowed traffic passes


## Key Takeaways
* NetworkPolicy = Pod-level firewall
* Controls Ingress & Egress
* Uses:
    * podSelector
    * namespaceSelector
    * ipBlock
* Rules are:
    * Allow-based
    * Everything else is denied
* Requires supported CNI (e.g., Calico)

## Pro Tip

Combine:
* Labels + namespaces + IP blocks

Build zero-trust networking inside your cluster


## Commands

### List Network Policies
```kubectl get networkpolicy```
or 
```kubectl get netpol```

### Describe a Network Policy
```kubectl describe netpol {name-of-network-policy}```


### Apply (Create/Update) a Network Policy
```kubectl apply -f {network-policy.yaml}```

--- 

# Problem

Create a network policy to allow egress traffic from the Internal application only to the payroll-service and db-service.

Use the spec given below. You might want to enable ingress traffic to the pod to test your rules in the UI.

Also, ensure that you allow egress traffic to DNS ports TCP and UDP (port 53) to enable DNS resolution from the internal pod.

## Solution
Solution manifest file for a network policy internal-policy as follows:
```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: internal-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      name: internal
  policyTypes:
  - Egress
  - Ingress
  ingress:
    - {}
  egress:
  - to:
    - podSelector:
        matchLabels:
          name: mysql
    ports:
    - protocol: TCP
      port: 3306

  - to:
    - podSelector:
        matchLabels:
          name: payroll
    ports:
    - protocol: TCP
      port: 8080

  - ports:
    - port: 53
      protocol: UDP
    - port: 53
      protocol: TCP
```

Explanation:

* Target Pods:

This policy applies to all pods in the default namespace with the label name: internal.

* Ingress:

All incoming traffic is allowed to these pods. This is typically needed for UI-based testing during labs.

*In production, you should restrict ingress to only trusted sources.*

* Egress:

Outbound traffic is restricted to:

    * Pods labeled name: mysql on TCP port 3306 (database service)
    * Pods labeled name: payroll on TCP port 8080 (payroll service)
    * Any destination on UDP/TCP port 53 (for DNS resolution, required for service discovery in Kubernetes)

* DNS Access:

DNS is handled by the kube-dns service, which listens on port 53 for both UDP and TCP:

```
root@controlplane:~> kubectl get svc -n kube-system 
NAME       TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE
kube-dns   ClusterIP   10.96.0.10   <none>        53/UDP,53/TCP,9153/TCP   18m

root@controlplane:~>
```