# Requests and Limits

## 🎯 Objective

* Understand **CPU and Memory Requests & Limits**.
* Learn how requests affect **Pod scheduling**.
* Understand how limits control **runtime resource usage**.
* Understand **CPU throttling** and **OOMKilled**.
* Practice identifying **insufficient resources** and resource-limit failures.
* Use `kubectl top` to observe CPU and memory usage.

---

# 1. What are Requests and Limits?

Resources are defined **per container** inside a Pod.

```yaml
resources:
  requests:
    cpu: "500m"
    memory: "256Mi"
  limits:
    cpu: "1"
    memory: "512Mi"
```

### Request

The amount of CPU/memory the container **requests for scheduling**.

The Scheduler uses requests to determine whether a Pod can fit on a Node. ([Kubernetes][2])

### Limit

The **maximum amount of the resource** the container is allowed to use.

At runtime:

* CPU limit → **CPU throttling**
* Memory limit → **OOM kill** when the kernel enforces the limit

([Kubernetes][1])

### 🧠 Easy Analogy

```text
Request → Minimum amount considered for scheduling
Limit   → Maximum allowed at runtime
```

---

# 2. CPU vs Memory Behavior

| Resource | Request             | Limit Exceeded             |
| -------- | ------------------- | -------------------------- |
| CPU      | Used for scheduling | CPU throttling             |
| Memory   | Used for scheduling | Container may be OOMKilled |

### CPU

If a container tries to use more CPU than its limit:

```text
CPU usage > CPU limit
        ↓
CPU throttling
        ↓
Container gets less CPU time
```

The container is **not normally killed just because it exceeds CPU usage**. ([Kubernetes][2])

### Memory

Memory is different:

```text
Memory usage > memory limit
        ↓
Kernel OOM handling
        ↓
Container may be terminated
        ↓
OOMKilled
```

Memory-limit enforcement is reactive; the container may not be killed at the exact instant it crosses the limit. ([Kubernetes][2])

---

# 3. Resource Units

## CPU

```text
1 CPU  = 1000m
500m   = 0.5 CPU
100m   = 0.1 CPU
```

## Memory

Common units:

```text
Mi = Mebibyte
Gi = Gibibyte
```

Prefer binary units such as `Mi` and `Gi` when specifying memory.

### 🧠 CKA Recall

```text
1000m = 1 CPU

Mi / Gi → binary memory units
M / G   → decimal memory units
```

---

# 4. Why Requests and Limits Matter

Without appropriate resource boundaries, a workload can consume a large amount of available Node resources.

```text
Pod consumes excessive memory
          ↓
Node memory pressure
          ↓
Kernel OOM handling
          ↓
Processes/containers may be killed
          ↓
Other Pods may be affected
```

Kubernetes therefore uses requests and limits to provide predictable scheduling and runtime resource control.

> **Important:** Requests and limits do not guarantee that a Node can never run out of resources. Node-level memory pressure can still cause Pod eviction or OOM-related behavior.

---

# 5. Requests vs Limits

|                      | Request                         | Limit                                                         |
| -------------------- | ------------------------------- | ------------------------------------------------------------- |
| Purpose              | Scheduling                      | Runtime control                                               |
| Used by              | Scheduler                       | Kubelet/container runtime/kernel                              |
| CPU                  | Scheduling requirement          | Throttling ceiling                                            |
| Memory               | Scheduling requirement          | OOM enforcement                                               |
| Can usage exceed it? | Yes, if resources are available | Generally no for CPU; memory overage may trigger OOM handling |

A container can use more than its **request** if the Node has available resources. ([Kubernetes][2])

Example:

```text
Request = 256Mi
Limit   = 512Mi

Actual usage:
256Mi → normal
400Mi → allowed
500Mi → allowed
>512Mi → may be OOMKilled
```

---

# 🧠 6. Must-Remember Cheat Sheet

```text
Requests = Scheduling
Limits   = Runtime control

CPU exceeds limit
→ Throttling

Memory exceeds limit
→ OOMKilled

Request < actual usage
→ Allowed if resources are available

Request > available Node resources
→ Pod cannot be scheduled
```

> **Request = what the Pod asks for**
> **Limit = how much the container is allowed to use**

---

# 🧪 7. Hands-on Lab

## Step 1 — Verify the Cluster

```bash
kubectl get nodes
kubectl get pods -A
```

Check available Node resources:

```bash
kubectl describe node <node-name>
```

Look at:

```text
Capacity
Allocatable
Allocated resources
```

> For scheduling, Kubernetes works with the Node's **allocatable resources**, not simply its raw capacity. ([Kubernetes][2])

---

# 8. Metrics Server

`kubectl top` displays recent CPU and memory usage for Nodes and Pods. It requires the **Metrics Server** to be installed and working. ([Kubernetes][3])

Check Metrics Server:

```bash
kubectl get pods -n kube-system
```

Look for:

```text
metrics-server
```

Test:

```bash
kubectl top nodes
kubectl top pods
```

For a specific namespace:

```bash
kubectl top pods -n <namespace>
```

For per-container usage:

```bash
kubectl top pod <pod-name> --containers
```

Metrics Server provides resource metrics used by `kubectl top` and resource-based autoscaling. ([Kubernetes][4])

---

# 9. Create a Namespace

```bash
kubectl create namespace mem-example
```

Verify:

```bash
kubectl get ns
```

---

# 10. Memory Request and Limit Lab

Create `mem-demo.yaml`:

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: mem-demo
  namespace: mem-example

spec:
  containers:
    - name: mem-demo-ctr
      image: polinux/stress

      resources:
        requests:
          memory: "100Mi"
        limits:
          memory: "200Mi"

      command: ["stress"]
      args:
        - "--vm"
        - "1"
        - "--vm-bytes"
        - "150M"
        - "--vm-hang"
        - "1"
```

Apply:

```bash
kubectl apply -f mem-demo.yaml
```

Check:

```bash
kubectl get pod -n mem-example
kubectl top pod -n mem-example
```

### What is happening?

```text
Memory request = 100Mi
Memory limit   = 200Mi
Stress usage   ≈ 150Mi

150Mi > request
150Mi < limit
        ↓
Pod can continue running
```

The container is allowed to use more than its request when resources are available, but it should remain within its configured limit. ([Kubernetes][2])

---

# 11. OOMKilled Lab

Now create a workload that attempts to use more memory than its limit.

Example:

```yaml
resources:
  requests:
    memory: "100Mi"
  limits:
    memory: "200Mi"
```

Stress command:

```yaml
args:
  - "--vm"
  - "1"
  - "--vm-bytes"
  - "300M"
  - "--vm-hang"
  - "1"
```

Apply:

```bash
kubectl apply -f mem-demo.yaml
```

Check:

```bash
kubectl get pod -n mem-example
```

Inspect the container:

```bash
kubectl describe pod mem-demo -n mem-example
```

Look for:

```text
Reason: OOMKilled
```

You can also check:

```bash
kubectl get pod mem-demo -n mem-example \
  -o jsonpath='{.status.containerStatuses[0].lastState.terminated.reason}'
```

Expected:

```text
OOMKilled
```

### Flow

```text
Container tries to use > 200Mi
             ↓
Memory limit reached
             ↓
Kernel OOM handling
             ↓
Container terminated
             ↓
OOMKilled
```

---

# 12. Insufficient Resources Lab

Now test the **scheduling side** of resource requests.

Create a Pod requesting more memory than any available Node can provide:

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: resource-demo

spec:
  containers:
    - name: nginx
      image: nginx

      resources:
        requests:
          memory: "100Gi"
          cpu: "10"
```

Apply:

```bash
kubectl apply -f resource-demo.yaml
```

Check:

```bash
kubectl get pod resource-demo
```

Expected:

```text
Pending
```

Inspect:

```bash
kubectl describe pod resource-demo
```

Check **Events**.

You should see a scheduling message indicating that available resources are insufficient.

### Flow

```text
Pod requests:
CPU    = 10
Memory = 100Gi
        ↓
Scheduler checks Nodes
        ↓
No Node satisfies the request
        ↓
Pod remains Pending
```

### 🧠 Important

> **Large request → Pod may remain Pending.**
> **Large runtime usage → container may be throttled/OOMKilled depending on the resource.**

---

# 13. CPU Throttling Lab

Create a Pod with a CPU limit:

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: cpu-demo

spec:
  containers:
    - name: cpu-demo
      image: polinux/stress

      resources:
        requests:
          cpu: "100m"
        limits:
          cpu: "200m"

      command: ["stress"]
      args:
        - "--cpu"
        - "1"
```

Apply:

```bash
kubectl apply -f cpu-demo.yaml
```

Check:

```bash
kubectl top pod cpu-demo
```

Inspect the Pod:

```bash
kubectl describe pod cpu-demo
```

### What happens?

The workload attempts to consume CPU, but its limit is:

```text
200m = 0.2 CPU
```

Therefore, the kernel restricts CPU time when the container reaches its CPU limit.

```text
Workload wants more CPU
        ↓
CPU limit = 200m
        ↓
CPU throttling
        ↓
Container continues running
```

CPU limits are enforced through throttling rather than terminating the container for CPU overuse. ([Kubernetes][1])

---

# 14. Cleanup

Delete the lab namespace:

```bash
kubectl delete namespace mem-example
```

Or delete individual resources:

```bash
kubectl delete pod mem-demo
kubectl delete pod cpu-demo
kubectl delete pod resource-demo
```

---

# 🔧 15. CKA Troubleshooting

## Pod is `Pending`

Start with:

```bash
kubectl describe pod <pod-name>
```

Check:

```text
Events
```

Common reason:

```text
Insufficient cpu
Insufficient memory
```

Then inspect Node resources:

```bash
kubectl describe node <node-name>
```

---

## Pod is `OOMKilled`

Check:

```bash
kubectl describe pod <pod-name>
```

Look for:

```text
Reason: OOMKilled
```

Then inspect:

```bash
kubectl get pod <pod-name> -o yaml
```

Check:

```yaml
resources:
  requests:
  limits:
```

Also check actual usage:

```bash
kubectl top pod <pod-name>
```

---

## CPU performance is unexpectedly low

Check:

```bash
kubectl top pod <pod-name>
```

Then inspect the CPU limit:

```bash
kubectl get pod <pod-name> -o yaml
```

A very low CPU limit can cause throttling even when the Node has spare CPU capacity. ([Kubernetes][1])

---

# 🧠 Final CKA Recall

```text
Request
→ Used by Scheduler

Limit
→ Runtime ceiling

CPU > limit
→ Throttling

Memory > limit
→ OOM handling / OOMKilled

Request > Node allocatable
→ Pod Pending

Request < actual usage
→ Usage can exceed request if resources are available

kubectl top
→ View recent CPU/Memory usage
```

### One-line memory rule

> **Requests decide WHERE the Pod can run; limits control HOW MUCH the container can consume.**


