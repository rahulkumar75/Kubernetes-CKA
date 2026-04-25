# ☸️ Kubernetes CKA Journey

<div align="center">

![Kubernetes](https://img.shields.io/badge/Kubernetes-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white)
![CKA](https://img.shields.io/badge/CKA-Certified%20Kubernetes%20Administrator-326CE5?style=for-the-badge)
![YAML](https://img.shields.io/badge/YAML-CB171E?style=for-the-badge&logo=yaml&logoColor=white)
![Days](https://img.shields.io/badge/Days%20Completed-30%2B-brightgreen?style=for-the-badge)

**A comprehensive day-by-day Kubernetes learning journey covering all CKA exam domains.**

</div>

---

## 📚 About This Repository

This repository documents my hands-on journey to master Kubernetes and prepare for the **Certified Kubernetes Administrator (CKA)** exam. Each folder represents a daily deep-dive into a specific Kubernetes concept — complete with YAML manifests, configuration files, and practical examples.

---

## 🗺️ Learning Path

### 🏗️ Core Architecture & Cluster Setup

| Day | Topic | Key Concepts |
|-----|-------|-------------|
| [DAY-6](./DAY-6-cluster/) | **Cluster Setup** | kubeadm, control plane, worker nodes, `config.yaml` |

---

### ⚙️ Workloads & Scheduling

| Day | Topic | Key Concepts |
|-----|-------|-------------|
| [Day-8](./Day-8-replica/) | **ReplicaSets** | Pod replication, self-healing, desired state |
| [Day-11](./Day-11-multiContainer/) | **Multi-Container Pods** | Sidecar, init containers, shared volumes |
| [Day12](./Day12-Daemonset/) | **DaemonSets** | Node-level daemons, log collectors, monitoring agents |
| [Day14](./Day14-taints/) | **Taints & Tolerations** | Node taints, pod tolerations, scheduling control |
| [Day15](./Day15-nodeAffinity/) | **Node Affinity** | Required/preferred affinity, node selectors |
| [Day16](./Day16-resources-limits/) | **Resource Limits** | CPU/memory requests & limits, LimitRange, ResourceQuota |
| [Day17](./Day17-autoscaling-hpa/) | **HPA - Autoscaling** | Horizontal Pod Autoscaler, metrics server |
| [Day18](./Day18-probes/) | **Probes** | Liveness, readiness, startup probes |

---

### 🌐 Services & Networking

| Day | Topic | Key Concepts |
|-----|-------|-------------|
| [Day-9](./Day-9-service/) | **Services** | ClusterIP, NodePort, LoadBalancer, Endpoints |
| [Day-10](./Day-10-namespace/) | **Namespaces** | Resource isolation, DNS, namespace-scoped resources |
| [Day26](./Day26-network-pol/) | **Network Policies** | Ingress/egress rules, pod selectors, namespace selectors |
| [Day27](./Day27/) | **Ingress** | Ingress controllers, routing, TLS termination |
| [Day28](./Day28/) | **DNS & CoreDNS** | Service discovery, custom DNS, CoreDNS config |

---

### 💾 Storage

| Day | Topic | Key Concepts |
|-----|-------|-------------|
| [Day29](./Day29/) | **Volumes & PVs** | PersistentVolume, PersistentVolumeClaim, StorageClass |
| [Day30](./Day30/) | **Volume Types** | emptyDir, hostPath, ConfigMap/Secret volumes |

---

### 🔐 Security

| Day | Topic | Key Concepts |
|-----|-------|-------------|
| [Day19](./Day19-configMap/) | **ConfigMaps & Secrets** | Environment injection, volume mounts, sensitive data |
| [Day20](./Day20-ssl/) | **SSL/TLS Basics** | Certificates, CA, public/private keys |
| [Day21](./Day21-TLS/) | **TLS in Kubernetes** | API server certificates, kubelet TLS, cert rotation |
| [Day22](./Day22-Auth/) | **Authentication** | kubeconfig, users, service accounts, tokens |
| [Day23](./Day23-RBAC/) | **RBAC** | Roles, RoleBindings, verbs, resources |
| [Day24](./Day24-cluster-role/) | **ClusterRoles** | Cluster-wide permissions, ClusterRoleBindings |
| [Day25](./Day25-service-acc/) | **Service Accounts** | Pod identity, RBAC for pods, token mounting |

---

### 🔧 Advanced Topics

| Day | Topic | Key Concepts |
|-----|-------|-------------|
| [Day31](./Day31/) | **etcd Backup & Restore** | `etcdctl snapshot`, disaster recovery |
| [Day32](./Day32/) | **Cluster Upgrade** | kubeadm upgrade, drain/uncordon, version skew |
| [Day33](./Day33/) | **Troubleshooting** | Pod failures, node issues, logging, `kubectl debug` |
| [Day34](./Day34/) | **Static Pods & Systemd** | Manifest path, kubelet config, system services |
| [Day35](./Day35/) | **CKA Exam Prep** | Practice scenarios, speed drills, imperative commands |

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
git clone https://github.com/rahulkumar75/Kubernetes-CKA.git
cd Kubernetes-CKA

# Apply a manifest
kubectl apply -f Day-8-replica/replicaset.yaml

# Explore a namespace setup
kubectl apply -f Day-10-namespace/
```

---

## 📋 CKA Exam Domain Coverage

| Domain | Weight | Days Covered |
|--------|--------|-------------|
| ☸️ Cluster Architecture, Installation & Configuration | 25% | Day 6, 31, 32 |
| 🌐 Services & Networking | 20% | Day 9, 10, 26, 27, 28 |
| 💾 Storage | 10% | Day 29, 30 |
| ⚙️ Workloads & Scheduling | 15% | Day 8, 11–18 |
| 🔐 Security | 15% | Day 19–25 |
| 🔧 Troubleshooting | 15% | Day 33, 34 |

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
Kubernetes-CKA/
├── DAY-6-cluster/          # Cluster bootstrap & kubeadm config
├── Day-8-replica/          # ReplicaSet manifests
├── Day-9-service/          # Service types
├── Day-10-namespace/       # Namespace isolation
├── Day-11-multiContainer/  # Multi-container pod patterns
├── Day12-Daemonset/        # DaemonSet examples
├── Day14-taints/           # Taints & tolerations
├── Day15-nodeAffinity/     # Node affinity rules
├── Day16-resources-limits/ # Resource management
├── Day17-autoscaling-hpa/  # HPA configuration
├── Day18-probes/           # Health probes
├── Day19-configMap/        # ConfigMaps & Secrets
├── Day20-ssl/              # SSL fundamentals
├── Day21-TLS/              # TLS in Kubernetes
├── Day22-Auth/             # Authentication
├── Day23-RBAC/             # RBAC policies
├── Day24-cluster-role/     # ClusterRoles
├── Day25-service-acc/      # Service accounts
├── Day26-network-pol/      # Network policies
├── Day27/ → Day35/         # Advanced & exam prep
└── config.yaml             # Cluster config reference
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
