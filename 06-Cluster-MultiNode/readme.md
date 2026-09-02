
# 6 — Install & Configure a Multi-Node Kubernetes Cluster with Kind

## Objective

Set up a **local multi-node Kubernetes cluster** using **Kind (Kubernetes IN Docker)** and learn how to:

* Install Kind and `kubectl`
* Create a Kubernetes cluster
* Create a multi-node cluster with control-plane and worker nodes
* Switch between Kubernetes contexts
* Verify cluster and node status
* Deploy a sample workload and identify which node runs it

---

# 1. Why Kind?

There are several ways to run Kubernetes locally:

* **Minikube**
* **Kind**
* **K3s**
* **K3d**
* Docker Desktop Kubernetes

**Kind** runs Kubernetes nodes as Docker containers. It is widely used for **local development, Kubernetes testing, labs, and CI environments**.

Official documentation:
[https://kind.sigs.k8s.io/docs/user/quick-start/](https://kind.sigs.k8s.io/docs/user/quick-start/)

---

# 2. Installation

### Install Kind

Follow the official installation guide:

[https://kind.sigs.k8s.io/docs/user/quick-start/#installation](https://kind.sigs.k8s.io/docs/user/quick-start/#installation)

Verify:

```bash
kind version
```

### Install kubectl

Install the Kubernetes CLI:

[https://kubernetes.io/docs/tasks/tools/](https://kubernetes.io/docs/tasks/tools/)

Verify:

```bash
kubectl version --client
```

---

# 3. Create a Basic Cluster

A simple Kind cluster can be created with:

```bash
kind create cluster --name cka-cluster
```

By default, Kind creates a **single-node cluster**, where the control plane also acts as the worker.

Verify:

```bash
kind get clusters
kubectl get nodes
```

---

# 4. Create a Multi-Node Cluster

For CKA practice, a multi-node cluster is more useful.

Create `config.yaml`:

```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4

nodes:
  - role: control-plane
  - role: worker
  - role: worker
```

Create the cluster:

```bash
kind create cluster \
  --name cka-cluster-3 \
  --config config.yaml
```

> **Note:** Kubernetes/Kind image versions change over time. Use a currently supported `kindest/node` image rather than blindly reusing an old image digest from a previous lab.

---

# 5. Understand the Cluster Structure

The resulting cluster looks like:

```text
                 Kubernetes Cluster
                       │
          ┌────────────┴────────────┐
          │                         │
   Control Plane                Workers
          │                  ┌──────┴──────┐
          │                  │             │
   API Server etcd      worker-1       worker-2
   Scheduler
   Controller Manager
```

In Kind, these nodes are actually **Docker containers**.

---

# 6. Verify the Cluster

### List Kind clusters

```bash
kind get clusters
```

### Check current Kubernetes context

```bash
kubectl config current-context
```

Expected:

```text
kind-cka-cluster-3
```

### List all contexts

```bash
kubectl config get-contexts
```

### Switch context

```bash
kubectl config use-context kind-cka-cluster-3
```

### Verify nodes

```bash
kubectl get nodes
```

Expected:

```text
NAME                          STATUS   ROLES           VERSION
cka-cluster-3-control-plane   Ready    control-plane   v1.35.0
cka-cluster-3-worker          Ready   <none>          v1.35.0
cka-cluster-3-worker2         Ready   <none>          v1.35.0
```

### Get detailed node information

```bash
kubectl get nodes -o wide
```

This shows:

* Internal IP
* OS image
* Kernel version
* Container runtime
* Kubernetes version

---

# 7. Verify Cluster Components

```bash
kubectl cluster-info
```

Or explicitly:

```bash
kubectl cluster-info --context kind-cka-cluster-3
```

Example:

```text
Kubernetes control plane is running at https://127.0.0.1:xxxxx
CoreDNS is running at ...
```

For deeper troubleshooting:

```bash
kubectl cluster-info dump
```

---

# 8. Check the Current Configuration

View the active cluster configuration:

```bash
kubectl config view --minify
```

This displays only the configuration for the **current context**.

To check the current namespace:

```bash
kubectl config view --minify | grep namespace
```

If no namespace is displayed, Kubernetes is using:

```text
default
```

> **Important:** Always confirm your current context before executing commands, especially when working with multiple clusters.

```bash
kubectl config current-context
```

---

# 9. Hands-On: Deploy a Workload

Create an Nginx Pod:

```bash
kubectl run nginx-worker-1 \
  --image=nginx \
  --restart=Never
```

Check the Pod:

```bash
kubectl get pods
```

Expected:

```text
NAME             READY   STATUS    RESTARTS   AGE
nginx-worker-1   1/1     Running   0          ...
```

---

# 10. Find Which Node Runs the Pod

Use:

```bash
kubectl get pods -o wide
```

Example:

```text
NAME             READY   STATUS    IP           NODE
nginx-worker-1   1/1     Running   10.244.1.2   cka-cluster-3-worker
```

This is an important Kubernetes concept:

```text
Pod
  ↓
Scheduled by Kubernetes Scheduler
  ↓
Worker Node
  ↓
Container Runtime
  ↓
Container
```

In this example, the scheduler placed `nginx-worker-1` on:

```text
cka-cluster-3-worker
```

---

# 11. Useful Verification Commands

| Purpose                | Command                                |
| ---------------------- | -------------------------------------- |
| List Kind clusters     | `kind get clusters`                    |
| Current context        | `kubectl config current-context`       |
| List contexts          | `kubectl config get-contexts`          |
| Switch context         | `kubectl config use-context <context>` |
| Cluster information    | `kubectl cluster-info`                 |
| List nodes             | `kubectl get nodes`                    |
| Detailed nodes         | `kubectl get nodes -o wide`            |
| List Pods              | `kubectl get pods`                     |
| Pod + node information | `kubectl get pods -o wide`             |
| Current configuration  | `kubectl config view --minify`         |

---

# 12. Final Lab Verification

Run:

```bash
kind get clusters

kubectl config current-context

kubectl get nodes

kubectl get nodes -o wide

kubectl get pods -o wide
```

Expected architecture:

```text
cka-cluster-3
│
├── control-plane
│     └── Kubernetes control-plane components
│
├── worker
│     └── nginx-worker-1
│
└── worker2
```

---

## Key CKA Takeaways

1. **Kind runs Kubernetes nodes as Docker containers.**
2. A default Kind cluster is generally **single-node**.
3. A Kind configuration file can create **control-plane + worker nodes**.
4. `kubectl` communicates with the cluster through the **current context**.
5. Always verify:

```bash
kubectl config current-context
```

before performing cluster operations.

6. `kubectl get pods -o wide` helps identify the **node on which a Pod is running**.
7. Kubernetes **Scheduler** decides which worker node should run a Pod.
8. `kubectl get nodes -o wide` provides additional node-level information.

### One command to remember

```bash
kubectl config use-context <context-name>
```

**Always know which Kubernetes cluster you are currently working on.**


