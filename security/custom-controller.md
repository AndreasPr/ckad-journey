## What is a Custom Controller in Kubernetes?

A **Custom Controller** is a program (usually written in Go) that extends Kubernetes by **watching custom resources** and continuously working to make the real world match the desired state defined in those resources.

It is a core building block of the **Kubernetes Operator pattern**.

---

## The Core Idea

Kubernetes itself is built on controllers.

For example:

* Deployment controller → ensures the correct number of Pods are running
* ReplicaSet controller → maintains desired replicas
* Node controller → monitors node health

A **custom controller** works the same way—but for **your own resources**.

---

## How It Fits Together

You typically use a custom controller with a **Custom Resource Definition (CRD)**.

### Flow:

1. **Define a Custom Resource (CRD)**

   * Example: `FlightTicket`

2. **User creates a resource**

   ```bash
   kubectl create -f flightticket.yaml
   ```

3. **Kubernetes stores it in etcd**

   * Nothing “real” happens yet

4. **Custom Controller detects the change**

   * Watches API Server for events (create/update/delete)

5. **Controller takes action**

   * Calls external API (e.g., book a flight)
   * Updates status
   * Retries if needed

---

## Controller Responsibilities

A custom controller continuously performs:

### 1. Watch

Listens for changes to resources:

* Create
* Update
* Delete

### 2. Compare (Desired vs Actual State)

Example:

* Desired: 2 flight tickets
* Actual: 0 booked tickets

### 3. Reconcile

Takes action to fix the difference:

* Calls external API
* Creates resources
* Updates status

This loop is called the **Reconciliation Loop**.

---

## Reconciliation Loop (Key Concept)

```
while true:
    desired_state = read from Kubernetes (etcd)
    actual_state = inspect real world / external system

    if desired_state != actual_state:
        take action to reconcile

    sleep / wait for next event
```

This loop is:

* Continuous
* Idempotent (safe to retry)
* Event-driven

---

## Example: Flight Ticket Controller

### Custom Resource

```yaml
kind: FlightTicket
spec:
  from: New York
  to: London
  number: 2
```

### What Controller Does

When it sees this:

1. Calls:

   ```
   https://book-flight.com/api
   ```

2. Sends:

   ```json
   {
     "from": "New York",
     "to": "London",
     "number": 2
   }
   ```

3. Updates resource status:

```yaml
status:
  bookingId: ABC123
  state: Confirmed
```

---

## Key Components of a Custom Controller

### 1. Informer

* Watches Kubernetes API
* Gets notified of changes

### 2. Work Queue

* Stores events to process
* Ensures retries

### 3. Reconciler Logic

* Your business logic
* Decides what to do

### 4. Client

* Communicates with Kubernetes API

---

## Example (Conceptual Go Code)

```go
func Reconcile(request Request) {
    resource := getResource(request)

    if resource.Spec.State != resource.Status.State {
        // Call external API
        result := callExternalAPI(resource.Spec)

        // Update status
        updateStatus(resource, result)
    }
}
```

---

## Important Characteristics

### 1. Idempotency

* Running multiple times should NOT break anything
* Same input → same result

### 2. Eventually Consistent

* Not immediate, but converges over time

### 3. Declarative

* Users define *what they want*
* Controller decides *how to achieve it*

---

## Where Controllers Run

Custom controllers usually run:

* As a **Pod inside the cluster**
* Often deployed as a **Deployment**

Example:

```yaml
kind: Deployment
spec:
  containers:
  - name: flight-controller
    image: myrepo/flight-controller
```

---

## What Happens on Delete?

When a resource is deleted:

1. Controller detects deletion event
2. Performs cleanup:

   * Cancel booking
   * Release resources
3. Optionally uses **Finalizers** to block deletion until cleanup completes

---

## Finalizers (Important Concept)

Prevent resource deletion until cleanup is done:

```yaml
metadata:
  finalizers:
    - flights.com/cleanup
```

Controller removes finalizer after cleanup → then deletion completes.

---

## Custom Controller vs Built-in Controller

| Feature                       | Built-in Controller | Custom Controller |
| ----------------------------- | ------------------- | ----------------- |
| Managed by Kubernetes         | Yes                 | No (you write it) |
| Works with built-in resources | Yes                 | No                |
| Works with CRDs               | No                  | Yes               |
| Business logic                | Generic             | Custom            |

---

## Real-World Use Cases

* Database provisioning (e.g., create DB when CR is created)
* Cloud resources (AWS, GCP, Azure)
* CI/CD pipelines
* Backup/restore automation
* Security policies
* SaaS integrations

---

## Operator Pattern

A **Kubernetes Operator** =
**CRD + Custom Controller + Domain Knowledge**

Example:

* Instead of manually managing PostgreSQL:

  * Create `PostgresCluster` resource
  * Operator handles:

    * backups
    * scaling
    * failover

---

## Summary

A **Custom Controller**:

* Watches Kubernetes resources (especially CRDs)
* Continuously reconciles desired state with actual state
* Automates real-world actions (APIs, infrastructure, etc.)
* Is the foundation of Kubernetes extensibility
