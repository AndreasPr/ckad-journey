<div align="center">
  
# ☸️ CKAD Journey
 
**Certified Kubernetes Application Developer — Exam Preparation Repository**
 
*Hands-on YAML manifests, kubectl references, and topic-by-topic Kubernetes labs built while studying for the CKAD certification*
 
[![Kubernetes](https://img.shields.io/badge/Kubernetes-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white)](https://kubernetes.io/)
[![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)](https://www.docker.com/)
[![Helm](https://img.shields.io/badge/Helm-0F1689?style=for-the-badge&logo=helm&logoColor=white)](https://helm.sh/)
[![YAML](https://img.shields.io/badge/YAML-CB171E?style=for-the-badge&logo=yaml&logoColor=white)](https://yaml.org/)
[![CKAD](https://img.shields.io/badge/CNCF-CKAD%20Prep-informational?style=for-the-badge&logo=cncf&logoColor=white)](https://www.cncf.io/certification/ckad/)
 
</div>

 
## 📌 Overview
 
This repository documents a **structured, topic-by-topic preparation journey for the CKAD (Certified Kubernetes Application Developer)** exam, issued by the Cloud Native Computing Foundation (CNCF).
 
Every folder maps to a discrete Kubernetes concept and contains hands-on YAML manifests, kubectl commands, and notes used to build real working knowledge — not just theory. The repo also includes three standalone reference guides covering imperative commands, container image building, and environment variable management, written in the exam-facing style of "here's what you need to know and be able to execute quickly."
 
> This repository demonstrates practical, production-relevant Kubernetes knowledge across application deployment, configuration management, networking, storage, security, and observability — covering the full scope of the CKAD exam curriculum.
 
---
 
## 🏅 CKAD Exam — What It Tests
 
The [CKAD certification](https://www.cncf.io/certification/ckad/) is a performance-based, hands-on exam administered by the Linux Foundation / CNCF. Candidates are tested in a live Kubernetes cluster environment and must demonstrate the ability to:
 
- Design, build, and deploy containerised applications to Kubernetes
- Configure and manage application resources, networking, and storage
- Apply security best practices at the workload level
- Troubleshoot running applications in a cluster
This repository covers **all major exam domains**.
 
---
 
## 🛠️ Skills & Technologies Demonstrated
 
| Category | Topics Covered |
|---|---|
| **Container Fundamentals** | Dockerfile authoring, ENTRYPOINT vs CMD, image build & push, Docker runtime |
| **Core Kubernetes Objects** | Pods, ReplicaSets, Deployments, Services, Namespaces, Jobs |
| **Configuration** | ConfigMaps, Secrets, Environment Variables |
| **Application Lifecycle** | Deployment strategies (rolling update, recreate), scaling, rollbacks |
| **Scheduling** | Node selectors, node affinity, taints & tolerations |
| **Networking** | Services (ClusterIP, NodePort, LoadBalancer), Network Policies |
| **Storage** | Persistent Volumes, Persistent Volume Claims |
| **Observability** | Readiness & liveness probes, logging, monitoring |
| **Security** | Security contexts, Service Accounts, RBAC-awareness |
| **Multi-container Patterns** | Sidecar, Init containers, Ambassador, Adapter patterns |
| **Package Management** | Helm charts (install, upgrade, rollback, template) |
| **Configuration Management** | Kustomize (overlays, bases, patches) |
| **kubectl Mastery** | Imperative commands, dry-run templating, output formatting, `kubectl explain` |
 
---
 
## 📂 Repository Structure
 
```
ckad-journey/
│
├── 📁 pods/                        # Pod creation, labels, selectors, annotations
├── 📁 replicaset/                  # ReplicaSet manifests and scaling
├── 📁 deployments/                 # Deployment definitions and management
├── 📁 deployment-strategy/         # Rolling update vs Recreate strategies
├── 📁 services/                    # ClusterIP, NodePort, LoadBalancer
├── 📁 namespaces/                  # Namespace isolation and resource quotas
│
├── 📁 configMaps/                  # ConfigMap creation and injection patterns
├── 📁 secrets/                     # Secret creation, base64 encoding, injection
├── 📁 resource-requirements/       # CPU/memory requests and limits
│
├── 📁 multi-container-pods/        # Sidecar, Init, Ambassador, Adapter patterns
├── 📁 readiness-liveness-probes/   # Health probes (HTTP, TCP, exec)
├── 📁 logging/                     # Application and container logging
├── 📁 monitoring/                  # Metrics and observability
│
├── 📁 node-selectors/              # Scheduling with node labels
├── 📁 node-affinity/               # Required and preferred affinity rules
├── 📁 taints-and-tolerations/      # Taint effects, tolerations in Pod specs
│
├── 📁 network-policy/              # Ingress/egress network isolation rules
├── 📁 storage/                     # PersistentVolumes, PVCs, StorageClasses
├── 📁 security-contexts/           # runAsUser, capabilities, readOnlyRootFilesystem
├── 📁 security/                    # Broader security configurations
├── 📁 service-account/             # ServiceAccount creation and Pod binding
│
├── 📁 jobs/                        # Jobs and CronJobs
├── 📁 Helm/                        # Helm chart usage and templating
├── 📁 kustomize/                   # Kustomize overlays and base patching
│
├── 📁 labs/                        # Practice lab exercises
├── 📁 tests/                       # Self-test scenarios and challenges
│
├── 📄 imperative-commands.md       # kubectl imperative command reference guide
├── 📄 build-modify-container-images.md  # Docker image building & Kubernetes overrides
└── 📄 environment-variables.md    # Env var patterns in Kubernetes
```
 
---
 
## 📖 Reference Guides
 
Three standalone markdown documents serve as quick-reference sheets for high-frequency exam topics:
 
### `imperative-commands.md`
A comprehensive guide to `kubectl` imperative commands covering Pods, Deployments, and Services. Includes the `--dry-run=client -o yaml` workflow for generating YAML templates quickly — a critical exam time-saving technique. Also covers output formatting (`-o json`, `-o wide`, `-o name`) and the `kubectl explain` documentation system.
 
### `build-modify-container-images.md`
Covers building and modifying Docker images from scratch (Ubuntu-based Flask app example), including a full Dockerfile walkthrough. Explains `ENTRYPOINT` vs `CMD` behaviour in Docker and how Kubernetes `command` and `args` fields map to and override them — a frequently-tested exam concept.
 
### `environment-variables.md`
Documents the different patterns for injecting environment variables into Kubernetes containers, including plain key-value pairs, ConfigMap references, Secret references, and field references.
 
---
 
## 🔑 Key Kubernetes Patterns Practised
 
**Deployment Strategy**
Rolling updates with `maxSurge` and `maxUnavailable`, Recreate strategy, rollback with `kubectl rollout undo`.
 
**Multi-Container Pods**
Sidecar (shared logging/proxy), Init containers (pre-start tasks), Ambassador (proxying), Adapter (output transformation).
 
**Health Probes**
HTTP GET, TCP socket, and exec-based readiness and liveness probes with configurable `initialDelaySeconds`, `periodSeconds`, and failure thresholds.
 
**Network Policies**
Ingress and egress isolation using `podSelector`, `namespaceSelector`, and `ipBlock` rules.
 
**Helm & Kustomize**
Package-based deployments with Helm (`helm install`, `helm upgrade`, `helm template`), and environment-specific patching with Kustomize overlays.
 
---
 
## 🚀 Using This Repository
 
This repo is primarily a **learning and reference artefact** rather than a deployable application. To explore any topic, navigate to the relevant folder and apply the manifests against a local or remote cluster.
 
### Prerequisites
 
- [`kubectl`](https://kubernetes.io/docs/tasks/tools/) configured against a cluster
- A local cluster (e.g. [minikube](https://minikube.sigs.k8s.io/), [kind](https://kind.sigs.k8s.io/), or [k3s](https://k3s.io/)) or remote access to a dev cluster
- [Docker](https://docs.docker.com/get-docker/) (for container image exercises)
- [Helm](https://helm.sh/docs/intro/install/) (for Helm folder exercises)
### Example — Applying a Manifest
 
```bash
# Clone the repository
git clone https://github.com/AndreasPr/ckad-journey.git
cd ckad-journey
 
# Apply any manifest
kubectl apply -f pods/
kubectl apply -f deployments/
 
# Generate a resource YAML without creating it (exam technique)
kubectl run nginx --image=nginx --dry-run=client -o yaml
 
# Clean up
kubectl delete -f pods/
```
 
