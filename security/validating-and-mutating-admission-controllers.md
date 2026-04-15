# Types of Admission Controllers in Kubernetes

---

## Overview

Admission Controllers are divided into two main types:

1. **Mutating Admission Controllers**
2. **Validating Admission Controllers**

They work together to enforce policies on Kubernetes objects.

---

## Execution Order

```

Request → Authentication → Authorization → Mutating → Validating → etcd

```

- **Mutating controllers run first**
- **Validating controllers run after**

### Why this order?

Because:
- Mutating controllers may **modify the request**
- Validating controllers then **verify the final state**

---

## Example Flow

1. A Pod creation request is submitted
2. `NamespaceAutoProvision` (mutating) creates the namespace if missing
3. `NamespaceExists` (validating) confirms the namespace exists
4. Request is accepted

---

# 1. Mutating Admission Controllers

---

## What are they?

Mutating Admission Controllers:
> Modify or enrich incoming requests before they are stored

---

## What can they do?

- Add default values
- Inject sidecar containers
- Add labels/annotations
- Modify security settings

---

## Examples

- **NamespaceAutoProvision** → creates namespace automatically  
- **DefaultStorageClass** → assigns default storage class  
- **MutatingAdmissionWebhook** → custom mutation logic  

---

## Example Use Cases

- Automatically add:
  - `env=prod` label to all Pods
- Inject:
  - logging sidecar container
- Enforce:
  - default resource limits

---

# 2. Validating Admission Controllers

---

## What are they?

Validating Admission Controllers:
> Validate requests and either allow or reject them

---

## What can they do?

- Enforce policies
- Block unsafe configurations
- Ensure compliance

---

## Examples

- **NamespaceExists** → ensures namespace exists  
- **ResourceQuota** → enforces resource limits  
- **ValidatingAdmissionWebhook** → custom validation logic  

---

## Example Use Cases

- Reject Pods:
  - using `latest` image tag  
  - running as root  
- Enforce:
  - required labels  
  - security constraints  

---

# Custom Admission Controllers (Webhooks)

---

## Why Use Webhooks?

Built-in controllers are limited.

If you need:
- Custom business logic
- Advanced validation
- Organization-specific policies

→ Use **Admission Webhooks**

---

## Types of Webhooks

1. **MutatingAdmissionWebhook**
2. **ValidatingAdmissionWebhook**

---

# MutatingAdmissionWebhook

---

## What is it?

An external service that:
> Intercepts requests and **modifies them dynamically**

---

## Example Use Cases

- Inject sidecar containers (e.g., logging, service mesh)
- Add default labels
- Modify resource configurations

---

## Example Flow

1. API server sends request to webhook
2. Webhook modifies object (JSON patch)
3. Modified object returns to API server
4. Request continues

---

# ValidatingAdmissionWebhook

---

## What is it?

An external service that:
> Validates requests and decides whether to allow or reject them

---

## Example Use Cases

- Enforce:
  - No public Docker images
  - No `latest` tags
- Validate:
  - Security policies
  - Required annotations

---

## Example Flow

1. API server sends request to webhook
2. Webhook evaluates request
3. Returns:
   - Allow → continue  
   - Deny → reject request  

---

# How to Implement Admission Webhooks

---

## Step 1: Deploy Webhook Server

You need a service that:
- Receives HTTP(S) requests from API server
- Processes admission review requests
- Returns responses (allow/deny or patch)

### Requirements:
- Must support **TLS**
- Must handle **AdmissionReview API**

---

## Step 2: Configure Admission Webhook

You register your webhook with Kubernetes using:

- `MutatingWebhookConfiguration`
- `ValidatingWebhookConfiguration`

---

# Example: External Webhook

```yaml
apiVersion: admissionregistration.k8s.io/v1
kind: ValidatingWebhookConfiguration
metadata:
  name: pod-policy-example.com

webhooks:
- name: pod-policy-example
  clientConfig:
    url: https://external-server.example.com
```

---

## When to Use This?

* Webhook hosted **outside Kubernetes**
* Example:

  * External policy engine

---

# Example: Internal Webhook (Recommended)

```yaml
apiVersion: admissionregistration.k8s.io/v1
kind: ValidatingWebhookConfiguration
metadata:
  name: pod-policy-example.com

webhooks:
- name: pod-policy-example
  clientConfig:
    service:
      namespace: webhook-namespace
      name: webhook-service
    caBundle: "base64-encoded-cert"

  rules:
  - apiGroups: [""]
    apiVersions: ["v1"]
    operations: ["CREATE"]
    resources: ["pods"]
    scope: "Namespaced"
```

---

## Explanation

### clientConfig.service

* Points to a Kubernetes Service
* API server calls this service internally

---

### caBundle

* Base64-encoded TLS certificate
* Ensures **secure communication**

---

### rules

Defines when the webhook is triggered:

* apiGroups → target API group
* apiVersions → API version
* operations → CREATE, UPDATE, DELETE
* resources → pods, deployments, etc
* scope → Namespaced or Cluster

---

# Important Concepts

---

## Failure Policy

Defines what happens if webhook fails:

* **Fail** → reject request (strict)
* **Ignore** → allow request (lenient)

---

## Side Effects

Indicates if webhook has side effects:

* None
* Some

Important for dry-run requests

---

## Timeout

* Webhooks must respond quickly
* Default timeout: ~10 seconds

---

# Summary

* Two types:

  * Mutating → modifies requests
  * Validating → approves/rejects requests

* Execution order:

  * Mutating → Validating

* Webhooks allow:

  * Custom logic
  * Advanced policy enforcement

* Common use cases:

  * Security enforcement
  * Compliance validation
  * Automatic configuration injection







# Scenarios
## Scenario 1
Create a TLS secret named `webhook-server-tls` in the `webhook-demo` namespace.
This secret will be used by the admission webhook server for secure communication over HTTPS.
We have already created below cert and key for webhook server which should be used to create secret.
Certificate : `/root/keys/webhook-server-tls.crt`
Key : `/root/keys/webhook-server-tls.key`

## Solution
Run below command:

```bash
kubectl -n webhook-demo create secret tls webhook-server-tls \
    --cert "/root/keys/webhook-server-tls.crt" \
    --key "/root/keys/webhook-server-tls.key"
```

## Scenario 2
Create the webhook deployment that will run the admission webhook server.
We have already provided the deployment manifest at:
`/root/webhook-deployment.yaml`

Create the deployement using this definition.

## Solution
Run command: `kubectl create -f /root/webhook-deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webhook-server
  namespace: webhook-demo
  labels:
    app: webhook-server
spec:
  replicas: 1
  selector:
    matchLabels:
      app: webhook-server
  template:
    metadata:
      labels:
        app: webhook-server
    spec:
      securityContext:
        runAsNonRoot: true
        runAsUser: 1234
      containers:
      - name: server
        image: stackrox/admission-controller-webhook-demo:latest
        imagePullPolicy: Always
        ports:
        - containerPort: 8443
          name: webhook-api
        volumeMounts:
        - name: webhook-tls-certs
          mountPath: /run/secrets/tls
          readOnly: true
      volumes:
      - name: webhook-tls-certs
        secret:
          secretName: webhook-server-tls
```

## Scenario 3
Create a service that exposes the webhook server so that the admission controller can communicate with it.
We have already provided the service manifest at:
`/root/webhook-service.yaml`
Create the service using this definition.


## Solution

Run command:`kubectl create -f /root/webhook-service.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: webhook-server
  namespace: webhook-demo
spec:
  selector:
    app: webhook-server
  ports:
    - port: 443
      targetPort: webhook-api
```


## Scenario 4
We have added the `MutatingWebhookConfiguration` under `/root/webhook-configuration.yaml`. Upon applying this configuration, which resources and actions will it impact?

## Solution
Pod with Create operations


```yaml
apiVersion: admissionregistration.k8s.io/v1
kind: MutatingWebhookConfiguration
metadata:
  name: demo-webhook
webhooks:
  - name: webhook-server.webhook-demo.svc
    clientConfig:
      service:
        name: webhook-server
        namespace: webhook-demo
        path: "/mutate"
      caBundle: <base64-encoded-cert>
    rules:
      - operations: [ "CREATE" ]
        apiGroups: [""]
        apiVersions: ["v1"]
        resources: ["pods"]
    admissionReviewVersions: ["v1beta1"]
    sideEffects: None
```

Also, deploy the `MutatingWebhookConfiguration` in `/root/webhook-configuration.yaml` by running:
```kubectl create -f /root/webhook-configuration.yaml```


## Notes
In the previous steps, you have set up and deployed a demo webhook with the following behaviors:

- Denies all requests for pods to run as root in a container if no `securityContext` is provided.
- Defaults: If `runAsNonRoot` is not set, the webhook automatically adds `runAsNonRoot: true` and sets the user ID to `1234`.
- Explicit root access: The webhook allows containers to run as root only if you explicitly set `runAsNonRoot: false` in the pod's `securityContext`.

In the next steps, you will find pod definition files for each scenario. Please deploy these pods using the provided definition files and validate the behavior of our webhook.


## Scenario 5
Deploy a pod that does not explicitly define a `securityContext`.
This will help verify that the webhook applies default values.
We have already provided the manifest:
`/root/pod-with-defaults.yaml`

## Solution

Run command: `kubectl apply -f /root/pod-with-defaults.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-defaults
  labels:
    app: pod-with-defaults
spec:
  restartPolicy: OnFailure
  containers:
    - name: busybox
      image: busybox
      command: ["sh", "-c", "echo I am running as user $(id -u)"]
```


## Scenario 6
Check the `securityContext` of the pod created in the previous step (pod-with-defaults).

## Solution
Even though we did not specify any values in the pod definition, the mutation webhook should have injected default values.

```yaml
controlplane ~ ➜  k get pod pod-with-defaults -o yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: "2026-04-05T03:51:03Z"
  generation: 1
  labels:
    app: pod-with-defaults
  name: pod-with-defaults
  namespace: default
  resourceVersion: "4331"
  uid: a15e7a5e-1bd8-42c9-bf04-b2cefd4236bc
spec:
  containers:
  - command:
    - sh
    - -c
    - echo I am running as user $(id -u)
    image: busybox
    imagePullPolicy: Always
    name: busybox
    resources: {}
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: kube-api-access-9w4gm
      readOnly: true
  dnsPolicy: ClusterFirst
  enableServiceLinks: true
  nodeName: controlplane
  preemptionPolicy: PreemptLowerPriority
  priority: 0
  restartPolicy: OnFailure
  schedulerName: default-scheduler
  securityContext:
    runAsNonRoot: true
    runAsUser: 1234
  serviceAccount: default
  serviceAccountName: default
```

## Scenario 7
Deploy pod with a `securityContext` explicitly allowing it to run as root
We have added pod definition file under
`/root/pod-with-override.yaml`

Validate securityContext after you deploy this pod.

## Solution
Run command:

`kubectl apply -f /root/pod-with-override.yaml`

then validate `securityContext` using the following command:

`kubectl get pod pod-with-override -o yaml | grep -A2 " securityContext:"`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-override
  labels:
    app: pod-with-override
spec:
  restartPolicy: OnFailure
  securityContext:
    runAsNonRoot: false
  containers:
    - name: busybox
      image: busybox
      command: ["sh", "-c", "echo I am running as user $(id -u)"]
```

## Scenario 8
Deploy a pod that specifies a conflicting `securityContext`.

- The pod requests to run with `runAsUser: 0` (root).
- But it does not explicitly set `runAsNonRoot: false`.

According to our webhook rules, this request should be denied.

We have already provided the manifest at: `/root/pod-with-conflict.yaml`


## Solution

Run command: `kubectl apply -f /root/pod-with-conflict.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-conflict
  labels:
    app: pod-with-conflict
spec:
  restartPolicy: OnFailure
  securityContext:
    runAsNonRoot: true
    runAsUser: 0
  containers:
    - name: busybox
      image: busybox
      command: ["sh", "-c", "echo I am running as user $(id -u)"]
```

Expected Outcome:
The admission webhook should reject this pod. You will see an error similar to:
```yaml
Error from server: error when creating "/root/pod-with-conflict.yaml": admission webhook "webhook-server.webhook-demo.svc" denied the request: runAsNonRoot specified, but runAsUser set to 0 (the root user)
```