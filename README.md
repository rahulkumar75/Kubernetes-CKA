# ☸️ Kubernetes CKA Journey
<img width="1774" height="887" alt="image" src="https://github.com/user-attachments/assets/0461ba0f-58c7-49f6-938a-6f1be988e3ba" />

---

## 📚 About This Repository

This repository documents my hands-on journey to master Kubernetes and prepare for the **Certified Kubernetes Administrator (CKA)** exam. Each folder represents a deep-dive into a specific Kubernetes (or supporting DevOps) concept — complete with YAML manifests, configuration files, and practical examples.

> 📁 **Note on folder naming:** Folders are organized by **topic number** (e.g. `08-Contorller-ReplicaSet`, `23-RBAC`). Numbers reflect the order topics were studied, so they aren't always sequential (some numbers are skipped or reused for related sub-topics) — always refer to the folder name for the actual subject.

## Goals
- Build strong Kubernetes fundamentals
- Gain production-level troubleshooting skills
- Prepare for the CKA certification
- Strengthen DevOps engineering knowledge

---

## 🗺️ Learning Path

### 🐳 Containers & Docker Fundamentals

| Folder | Topic | Key Concepts |
|--------|-------|---------------|
| [01-Docker](./01-Docker/) | **Docker Basics** | Images, containers, core Docker commands |
| [02-DockerImages](./02-DockerImages/) | **Docker Images** | Dockerfile, layers, image builds |
| [03-MultiStage-Docker](./03-MultiStage-Docker/) | **Multi-Stage Builds** | Smaller images, build vs runtime stages |
| [53-Dockerd-Setup](./53-Dockerd-Setup/) | **CRI-Dockerd Setup** | Container runtime installation for Kubernetes |

---

### 🏗️ Core Architecture & Cluster Setup

| Folder | Topic | Key Concepts |
|--------|-------|---------------|
| [04-Kubernetes-Architecture](./04-Kubernetes-Architecture/) | **Kubernetes Architecture** | Control plane, kubelet, kube-proxy, etcd |
| [06-Cluster-MultiNode](./06-Cluster-MultiNode/) | **Cluster Setup** | kubeadm, KIND, control plane, worker nodes |
| [27-MultiNodeCluster](./27-MultiNodeCluster/) | **Multi-Node Cluster Lab** | Hands-on multi-node cluster bring-up |
| [34-UpgradeCluster](./34-UpgradeCluster/) | **Cluster Upgrade** | kubeadm upgrade, drain/uncordon, version skew |

---

### ⚙️ Workloads & Scheduling

| Folder | Topic | Key Concepts |
|--------|-------|---------------|
| [08-Contorller-ReplicaSet](./08-Contorller-ReplicaSet/) | **Controllers & ReplicaSets** | Pod replication, self-healing, desired state |
| [11-MultiContainer](./11-MultiContainer/) | **Multi-Container Pods** | Sidecar, init containers, shared volumes |
| [12-Daemonset-CronJobs](./12-Daemonset-CronJobs/) | **DaemonSets & CronJobs** | Node-level daemons, scheduled jobs |
| [14-Taints-Tolerations](./14-Taints-Tolerations/) | **Taints & Tolerations** | Node taints, pod tolerations, scheduling control |
| [15-NodeAffinity](./15-NodeAffinity/) | **Node Affinity** | Required/preferred affinity, node selectors |
| [16-Resources-Limits](./16-Resources-Limits/) | **Resource Limits** | CPU/memory requests & limits, metrics-server |
| [17-Autoscaling-HPA](./17-Autoscaling-HPA/) | **HPA - Autoscaling** | Horizontal Pod Autoscaler, metrics server |
| [18-Probes](./18-Probes/) | **Probes** | Liveness, readiness, startup probes |
| [45-StatefulSet](./45-StatefulSet/) | **StatefulSets** | Stable identity, ordered deployment, MongoDB example |
| [46-PodPriority-Preemption](./46-PodPriority-Preemption/) | **Pod Priority & Preemption** | PriorityClasses, scheduling preemption |

---

### 🌐 Services & Networking

| Folder | Topic | Key Concepts |
|--------|-------|---------------|
| [09-Services](./09-Services/) | **Services** | ClusterIP, NodePort, LoadBalancer, Endpoints |
| [10-Namespaces](./10-Namespaces/) | **Namespaces** | Resource isolation, DNS, namespace-scoped resources |
| [26-NetworkPolicy](./26-NetworkPolicy/) | **Network Policies** | Ingress/egress rules, pod & namespace selectors |
| [30-DNS](./30-DNS/) | **DNS** | Service discovery basics |
| [31-CoreDNS](./31-CoreDNS/) | **CoreDNS** | Custom DNS, CoreDNS configuration |
| [32-K8s-Networking](./32-K8s-Networking/) | **K8s Networking Deep Dive** | tcpdump labs, networking troubleshooting |
| [33-Ingress](./33-Ingress/) | **Ingress** | Ingress controllers, routing, TLS termination |
| [47-GatewayAPI](./47-GatewayAPI/) | **Gateway API** | URL rewrite, traffic splitting & weighted routing |

---

### 💾 Storage

| Folder | Topic | Key Concepts |
|--------|-------|---------------|
| [29-PV-PVC-SC](./29-PV-PVC-SC/) | **Volumes & PVs** | PersistentVolume, PersistentVolumeClaim, StorageClass |
| [52-StorageClass](./52-StorageClass/) | **StorageClass** | Dynamic provisioning basics |
| [52-StorageClass-Dynamic-Vol-Pro](./52-StorageClass-Dynamic-Vol-Pro/) | **Dynamic Volume Provisioning** | AWS EBS StorageClass, local provisioning labs |

---

### 🔐 Security

| Folder | Topic | Key Concepts |
|--------|-------|---------------|
| [19-ConfigMaps](./19-ConfigMaps/) | **ConfigMaps & Secrets** | Environment injection, volume mounts, sensitive data |
| [20-SSL](./20-SSL/) | **SSL Basics** | Certificates, CA, public/private keys |
| [21-TLS](./21-TLS/) | **TLS in Kubernetes** | CSRs, API server certificates, cert rotation |
| [22-Auth](./22-Auth/) | **Authentication** | kubeconfig, users, service accounts, tokens |
| [23-RBAC](./23-RBAC/) | **RBAC** | Roles, RoleBindings, verbs, resources |
| [24-Cluster-Role](./24-Cluster-Role/) | **ClusterRoles** | Cluster-wide permissions, ClusterRoleBindings |
| [25-ServiceAccount](./25-ServiceAccount/) | **Service Accounts** | Pod identity, RBAC for pods, token mounting |
| [51-AddmissionControllers](./51-AddmissionControllers/) | **Admission Controllers** | Validating/mutating webhooks, kube-apiserver config |
| [54-Pod-Security-Standard](./54-Pod-Security-Standard/) | **Pod Security Standards** | Restricted/baseline/privileged profiles |

---

### 🧩 Extensibility & Advanced Topics

| Folder | Topic | Key Concepts |
|--------|-------|---------------|
| [40-JSON](./40-JSON/) | **JSON for Kubernetes** | JSON manifests, `kubectl` output formats |
| [49-CRD-CR](./49-CRD-CR/) | **CRDs & Custom Resources** | API extension, architecture & flow |
| [50-Operators](./50-Operators/) | **Operators** | Operator pattern, fundamentals, implementation |

---

### 🛠️ Cluster Operations & Troubleshooting

| Folder | Topic | Key Concepts |
|--------|-------|---------------|
| [35-ETCD-Backup-Restore](./35-ETCD-Backup-Restore/) | **etcd Backup & Restore** | `etcdctl snapshot`, disaster recovery |
| [36-Logging-Monitoring](./36-Logging-Monitoring/) | **Logging & Monitoring** | Cluster observability basics |
| [38-Troubleshoot-ControlPlane](./38-Troubleshoot-ControlPlane/) | **Control Plane Troubleshooting** | Diagnosing control plane failures |
| [39-Troubleshoot-WorkerNode](./39-Troubleshoot-WorkerNode/) | **Worker Node Troubleshooting** | Diagnosing kubelet/node issues |

---

### 🧰 Tooling & Reference

| Folder | Topic | Key Concepts |
|--------|-------|---------------|
| [AWS](./AWS/) | **AWS Configuration** | Cloud provider setup notes |
| [Git](./Git/) | **Git Cheatsheet** | Common Git commands & workflow |
| [Helm-Chart](./Helm-Chart/) | **Helm** | Helm runbook, workflow, publishing charts |
| [Mind-Maps](./Mind-Maps/) | **Mind Maps** | Visual summaries of key topics (DNS, RBAC, CRDs, etc.) |
| [Others](./Others/) | **Misc** | Miscellaneous notes |

---

## 🚀 Quick Start

### Prerequisites

```bash
# Tools you'll need
kubectl version --client
kubeadm version
minikube version    # or kind / k3s for local clusters
```

### Clone & Explore

```bash
git clone https://github.com/rahulkumar75/Kubernetes-CKA-Journey.git
cd Kubernetes-CKA-Journey

# Apply a manifest
kubectl apply -f 08-Contorller-ReplicaSet/rs.yaml

# Explore a namespace setup
kubectl apply -f 10-Namespaces/ns.yaml
```

---

## 📋 CKA Exam Domain Coverage

| Domain | Weight | Folders Covered |
|--------|--------|------------------|
| ☸️ Cluster Architecture, Installation & Configuration | 25% | `04`, `06`, `27`, `34`, `53` |
| 🌐 Services & Networking | 20% | `09`, `10`, `26`, `30`–`33`, `47` |
| 💾 Storage | 10% | `29`, `52` |
| ⚙️ Workloads & Scheduling | 15% | `08`, `11`–`18`, `45`, `46` |
| 🔐 Security | 15% | `19`–`25`, `51`, `54` |
| 🔧 Troubleshooting | 15% | `35`, `36`, `38`, `39` |

---

## ⚡ Handy kubectl Commands

```bash
# Pod operations
kubectl run nginx --image=nginx --dry-run=client -o yaml
kubectl exec -it <pod> -- /bin/sh
kubectl logs <pod> --previous

# Deployment
kubectl create deployment app --image=nginx --replicas=3
kubectl scale deployment app --replicas=5
kubectl rollout undo deployment/app

# RBAC
kubectl create role pod-reader --verb=get,list,watch --resource=pods
kubectl create rolebinding rb --role=pod-reader --user=dev

# Quick resource check
kubectl top nodes
kubectl top pods -A

# Helpful aliases (add to ~/.bashrc)
alias k=kubectl
alias kgp='kubectl get pods'
alias kgs='kubectl get svc'
export do='--dry-run=client -o yaml'
```

---

## 📁 Repository Structure

```
Kubernetes-CKA-Journey/
├── 01-Docker/                         # Docker basics
├── 02-DockerImages/                   # Docker image builds
├── 03-MultiStage-Docker/              # Multi-stage Docker builds
├── 04-Kubernetes-Architecture/        # Cluster architecture overview
├── 06-Cluster-MultiNode/              # Cluster bootstrap (kubeadm/KIND)
├── 08-Contorller-ReplicaSet/          # Controllers & ReplicaSet manifests
├── 09-Services/                       # Service types
├── 10-Namespaces/                     # Namespace isolation
├── 11-MultiContainer/                 # Multi-container pod patterns
├── 12-Daemonset-CronJobs/             # DaemonSet & CronJob examples
├── 14-Taints-Tolerations/             # Taints & tolerations
├── 15-NodeAffinity/                   # Node affinity rules
├── 16-Resources-Limits/               # Resource management
├── 17-Autoscaling-HPA/                # HPA configuration
├── 18-Probes/                         # Health probes
├── 19-ConfigMaps/                     # ConfigMaps & Secrets
├── 20-SSL/                            # SSL fundamentals
├── 21-TLS/                            # TLS in Kubernetes
├── 22-Auth/                           # Authentication
├── 23-RBAC/                           # RBAC policies
├── 24-Cluster-Role/                   # ClusterRoles
├── 25-ServiceAccount/                 # Service accounts
├── 26-NetworkPolicy/                  # Network policies
├── 27-MultiNodeCluster/               # Multi-node cluster lab
├── 29-PV-PVC-SC/                      # PV, PVC & StorageClass
├── 30-DNS/                            # DNS basics
├── 31-CoreDNS/                        # CoreDNS configuration
├── 32-K8s-Networking/                 # Networking deep dive & tcpdump labs
├── 33-Ingress/                        # Ingress controllers & routing
├── 34-UpgradeCluster/                 # Cluster upgrade
├── 35-ETCD-Backup-Restore/            # etcd backup & restore
├── 36-Logging-Monitoring/             # Logging & monitoring
├── 38-Troubleshoot-ControlPlane/      # Control plane troubleshooting
├── 39-Troubleshoot-WorkerNode/        # Worker node troubleshooting
├── 40-JSON/                           # JSON manifests
├── 45-StatefulSet/                    # StatefulSets
├── 46-PodPriority-Preemption/         # Pod priority & preemption
├── 47-GatewayAPI/                     # Gateway API labs
├── 49-CRD-CR/                         # CRDs & Custom Resources
├── 50-Operators/                      # Operator pattern
├── 51-AddmissionControllers/          # Admission controllers
├── 52-StorageClass/                   # StorageClass basics
├── 52-StorageClass-Dynamic-Vol-Pro/   # Dynamic volume provisioning
├── 53-Dockerd-Setup/                  # CRI-Dockerd setup
├── 54-Pod-Security-Standard/          # Pod Security Standards
├── AWS/                                # AWS configuration notes
├── Git/                                # Git cheatsheet
├── Helm-Chart/                         # Helm runbook & workflows
├── Mind-Maps/                          # Topic mind maps
├── Others/                              # Misc notes
└── config.yaml                         # Cluster config reference
```

---

## 🎯 CKA Exam Tips

> 💡 **Imperative over declarative** — In the exam, use `kubectl create/run` with `--dry-run=client -o yaml` to generate YAML fast.

> ⏱️ **Time management** — Skip hard questions, flag them, and come back. Each question shows its weight.

> 📖 **Bookmark these docs** — `kubernetes.io/docs` is allowed! Know where to find: Pod spec, RBAC, NetworkPolicy, PV/PVC examples.

> 🖥️ **Use aliases** — Set `alias k=kubectl` and `export do='--dry-run=client -o yaml'` at the start of the exam.

> 🔍 **`kubectl explain`** — Your best friend for field lookup: `kubectl explain pod.spec.containers.livenessProbe`

---

## 🔗 Resources

- 📖 [Official Kubernetes Docs](https://kubernetes.io/docs/)
- 🎓 [CNCF CKA Curriculum](https://github.com/cncf/curriculum)
- 🧪 [Killer.sh CKA Simulator](https://killer.sh/cka)
- 📝 [Kubernetes the Hard Way](https://github.com/kelseyhightower/kubernetes-the-hard-way)
- 🎥 [KodeKloud CKA Course](https://kodekloud.com/courses/certified-kubernetes-administrator-cka/)

---

## 👤 Author

**Rahul Kumar**
- GitHub: [@rahulkumar75](https://github.com/rahulkumar75)

---

<div align="center">

⭐ **Star this repo** if it helped you on your CKA journey!

</div>
