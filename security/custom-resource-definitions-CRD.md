# Custom Resource Definitions (CRDs) in Kubernetes

---

## What are Custom Resource Definitions (CRDs)?

A **Custom Resource Definition (CRD)** allows you to extend the Kubernetes API by defining your own custom resource types.

In simple terms:

> CRDs let you create your own Kubernetes objects, just like Pods, Deployments, or Services.

---

## Why Use CRDs?

Kubernetes is designed to be extensible. With CRDs, you can:

- Model domain-specific concepts (e.g., FlightTicket, Database, Backup)
- Use `kubectl` to manage custom resources
- Store custom objects in **etcd**
- Build full automation using controllers

---

## Example Use Case

You want to define a new resource:

```yaml
apiVersion: flights.com/v1
kind: FlightTicket
metadata:
  name: my-flight-ticket
spec:
  from: New York
  to: London
  number: 2
````

Then run:

```bash
kubectl create -f flightticket.yaml
kubectl get flightticket
kubectl delete -f flightticket.yaml
```

This creates a **Custom Resource (CR)** stored in etcd.

---

## Important Concept

> By default, CRDs ONLY store data.
> They do NOT perform any real-world action.

---

## How Do We Add Real Behavior?

To make this actually **book a flight**, we need:

### A Controller

A **controller** watches for changes to your custom resource and performs actions.

---

## Example Flow

1. User creates a `FlightTicket`
2. Kubernetes stores it in etcd
3. Custom controller detects the new object
4. Controller calls:

```
https://book-flight.com/api
```

5. Flight is booked

---

# Step 1: Define a CRD

---

## flightticket-custom-definition.yaml

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: flighttickets.flights.com

spec:
  scope: Namespaced
  group: flights.com

  names:
    kind: FlightTicket
    singular: flightticket
    plural: flighttickets
    shortNames:
      - ft

  versions:
  - name: v1
    served: true
    storage: true

    schema:
      openAPIV3Schema:
        type: object
        properties:
          spec:
            type: object
            properties:
              from:
                type: string
              to:
                type: string
              number:
                type: integer
                minimum: 1
                maximum: 10
```

---

## Explanation

---

### group

```yaml
group: flights.com
```

* Defines API group
* Full API becomes:

```
flights.com/v1
```

---

### names

```yaml
kind: FlightTicket
singular: flightticket
plural: flighttickets
shortNames: [ft]
```

* `kind` → object type
* `plural` → used in API (`kubectl get flighttickets`)
* `singular` → display name
* `shortNames` → shortcut (`kubectl get ft`)

---

### scope

```yaml
scope: Namespaced
```

* Namespaced → exists in a namespace
* Cluster → cluster-wide resource

---

### versions

```yaml
- name: v1
  served: true
  storage: true
```

* `served` → API is available
* `storage` → stored in etcd

---

### schema (Validation)

```yaml
openAPIV3Schema:
```

* Defines structure of the resource
* Enforces:

  * Data types
  * Required fields
  * Constraints

---

## Example Validation Rule

```yaml
number:
  type: integer
  minimum: 1
  maximum: 10
```

* Prevents invalid values

---

# Step 2: Create the CRD

```bash
kubectl create -f flightticket-custom-definition.yaml
```

---

## Verify

```bash
kubectl get crd
```

You should see:

```
flighttickets.flights.com
```

---

# Step 3: Create a Custom Resource

---

## flightticket.yaml

```yaml
apiVersion: flights.com/v1
kind: FlightTicket
metadata:
  name: my-flight-ticket
spec:
  from: New York
  to: London
  number: 2
```

---

## Create Resource

```bash
kubectl create -f flightticket.yaml
```

---

## View Resource

```bash
kubectl get flightticket
```

or

```bash
kubectl get ft
```

---

## Delete Resource

```bash
kubectl delete -f flightticket.yaml
```

---

# CRD vs Custom Resource

---

| Concept              | Description               |
| -------------------- | ------------------------- |
| CRD                  | Defines new resource type |
| Custom Resource (CR) | Instance of that type     |

---

# Controller (Critical Component)

---

## What is a Controller?

A controller is a program (usually written in Go) that:

* Watches CRs using Kubernetes API
* Performs actions based on desired state

---

## Example Responsibilities

* Call external APIs
* Create/update/delete resources
* Maintain system state

---

## Example

`flightticket_controller.go`:

* Watches FlightTicket resources
* Calls booking API
* Updates status

---

# Key Concept: Desired State

---

CRDs define:

> What you WANT

Controllers ensure:

> What ACTUALLY happens

---



## Commands
```bash
kubectl get crd {custom-resource-definition-name}
```

```bash
kubectl describe crd {custom-resource-definition-name}
```



# Real-World Examples of CRDs

---

Kubernetes ecosystem uses CRDs heavily:

* Prometheus → ServiceMonitor
* Istio → VirtualService
* ArgoCD → Application
* Cert-Manager → Certificate

---

# Summary

---

* CRDs extend Kubernetes API
* Custom Resources store data in etcd
* Controllers add real behavior
* Together, they form the **Operator pattern**
* Enable Kubernetes to manage complex systems




# Scenarios
## Scenario 1
We have provided an incomplete `Custom Resource Definition (CRD)` manifest located at `/root/crd.yaml`.

Your task is to complete this file to define a namespaced CRD named `internals.datasets.andreas.com`.

Please ensure you adhere to the following specifications:

* The CRD must belong to the group `datasets.andreas.com`.
* The scope of the CRD should be set to `Namespaced`.
* The version must be `v1`, and it should be marked as both `served: true` and `storage: true`.

Additionally, include a basic OpenAPI v3 schema for the CRD under the spec section with the following fields:
* `internalLoad` (string)
* `range` (integer)
* `percentage` (string)

Once you have created the CRD, utilize the provided `/root/custom.yaml` file to create a corresponding custom resource.

## Solution
The solution file for `crd.yaml` is pasted below:

```yaml
---
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: internals.datasets.andreas.com 
spec:
  group: datasets.andreas.com
  versions:
    - name: v1
      served: true
      storage: true
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              properties:
                internalLoad:
                  type: string
                range:
                  type: integer
                percentage:
                  type: string
  scope: Namespaced 
  names:
    plural: internals
    singular: internal
    kind: Internal
    shortNames:
    - int
```

Also make sure to create custom resource using `kubectl create -f custom.yaml` after correcting and creating CRD.





