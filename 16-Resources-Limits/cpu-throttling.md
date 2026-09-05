# 🧪 CPU Throttling Demo

## 🎯 Objective

* Understand the difference between **CPU requests and limits**.
* Demonstrate **CPU throttling** using a CPU-intensive workload.
* Observe how a CPU limit affects container CPU usage.
* Compare a **limited** container with an **unlimited** container.
* Understand why CPU throttling is different from `OOMKilled`.

---

# 1. Core Concept

```text
CPU Request → Used for scheduling
CPU Limit   → Maximum CPU usage enforced at runtime
```

If a container tries to use more CPU than its limit:

```text
Container wants more CPU
        ↓
CPU limit reached
        ↓
CPU throttling
        ↓
Container continues running
```

> **CPU exceeding limit → throttling, not container termination.**

---

# 2. Create a CPU Stress Pod

Create `cpu-demo.yaml`:

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: cpu-demo

spec:
  containers:
    - name: stress
      image: polinux/stress

      resources:
        requests:
          cpu: "100m"
        limits:
          cpu: "200m"

      command: ["stress"]
      args:
        - "--cpu"
        - "4"
```

### What does this mean?

```text
CPU request = 100m = 0.1 CPU
CPU limit   = 200m = 0.2 CPU

Stress workload
→ tries to consume 4 CPU
```

The workload wants significantly more CPU than the configured limit allows.

```text
Requested by workload → 4 CPU
Allowed by limit      → 0.2 CPU
                         ↓
                    Throttling
```

---

# 3. Apply the Pod

```bash
kubectl apply -f cpu-demo.yaml
```

Check:

```bash
kubectl get pod cpu-demo
```

Wait until:

```text
STATUS
Running
```

---

# 4. Observe CPU Usage

Use:

```bash
kubectl top pod cpu-demo
```

Or:

```bash
kubectl top pod cpu-demo --containers
```

You should see CPU usage around the configured limit:

```text
NAME        CPU(cores)
cpu-demo    ~200m
```

The exact value can fluctuate because metrics are sampled and reported periodically.

### Important

Do **not** expect `kubectl describe pod` to show a message such as:

```text
CPU throttled
```

CPU throttling is not normally reported there as a simple Pod event.

---

# 5. Why You May Not See a Clear CPU Spike

A common mistake is checking only:

```bash
kubectl top nodes
```

Node-level metrics are aggregated across workloads and can make throttling difficult to see.

Prefer:

```bash
kubectl top pod cpu-demo
```

You can also check:

```bash
kubectl top pod cpu-demo --containers
```

### Another important point

`kubectl top` shows **observed CPU usage**, not the amount of CPU the application is trying to consume.

Therefore:

```text
Application wants → 4 CPU
Container limit   → 200m
Observed usage    → around 200m
```

The workload can be CPU-intensive while the reported usage remains near the limit.

---

# 6. Compare With a Pod Having No CPU Limit

Create another Pod:

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: cpu-unlimited

spec:
  containers:
    - name: stress
      image: polinux/stress

      resources:
        requests:
          cpu: "100m"

      command: ["stress"]
      args:
        - "--cpu"
        - "4"
```

Notice:

```yaml
limits:
  cpu: ...
```

is not defined.

Apply:

```bash
kubectl apply -f cpu-unlimited.yaml
```

Check both Pods:

```bash
kubectl top pods
```

You can sort by CPU:

```bash
kubectl top pods --sort-by=cpu
```

Conceptually:

| Pod             | CPU Request | CPU Limit | Expected Behavior               |
| --------------- | ----------: | --------: | ------------------------------- |
| `cpu-demo`      |        100m |      200m | CPU usage constrained/throttled |
| `cpu-unlimited` |        100m |  No limit | Can use more CPU if available   |

> A container without a CPU limit can consume available CPU, subject to Node capacity and competition from other workloads.

---

# 7. CPU Throttling vs Memory OOMKilled

This is an important CKA distinction:

| Resource | When usage reaches limit |
| -------- | ------------------------ |
| CPU      | **Throttling**           |
| Memory   | **OOMKilled** may occur  |

### CPU

```text
CPU limit reached
      ↓
Throttling
      ↓
Container continues running
```

### Memory

```text
Memory limit exceeded
      ↓
OOM handling
      ↓
Container may be terminated
      ↓
OOMKilled
```

---

# 8. Real-Time Impact of CPU Throttling

CPU throttling may not crash an application, but it can affect performance.

Example:

```text
API Pod
CPU limit = 200m
      ↓
Traffic increases
      ↓
Application needs more CPU
      ↓
CPU limit reached
      ↓
Throttling
      ↓
Requests take longer
      ↓
Higher latency
```

Typical symptoms:

* Slow application response
* Increased request latency
* Lower throughput
* Timeouts under heavy load

### 🧠 Production Insight

> **CPU throttling is often a performance problem, not a crash problem.**

---

# 9. Experiment: Remove the CPU Limit

Modify:

```yaml
resources:
  requests:
    cpu: "100m"
```

Remove:

```yaml
limits:
  cpu: "200m"
```

Recreate the Pod:

```bash
kubectl delete pod cpu-demo
kubectl apply -f cpu-demo.yaml
```

Then observe:

```bash
kubectl top pod cpu-demo
```

If the Node has available CPU, the workload can consume more CPU than it could when limited to `200m`.

---

# 🔧 10. Troubleshooting

### Pod is not showing high CPU

Check:

```bash
kubectl get pod cpu-demo
kubectl top pod cpu-demo
```

Then verify the configuration:

```bash
kubectl get pod cpu-demo -o yaml
```

Check:

```yaml
resources:
  requests:
  limits:
```

Also verify that Metrics Server is working:

```bash
kubectl top nodes
```

If `kubectl top` does not work, check:

```bash
kubectl get pods -n kube-system
```

for `metrics-server`.

---

### Pod is Running but CPU appears low

Remember:

```text
Stress workload
      ↓
tries to consume CPU
      ↓
CPU limit restricts it
      ↓
reported usage stays around limit
```

Also remember that `kubectl top` is based on sampled metrics, so values are not an exact instantaneous measurement.

---

# 🧹 11. Cleanup

```bash
kubectl delete pod cpu-demo cpu-unlimited
```

---
# 🚀 Final Understanding

## With limits

- Controlled
- Stable
- Predictable

## Without limits

- Can consume full CPU
- Can affect other pods

# 💯 CKA Exam Tip

👉 If question says:

> Pod is slow but not crashing
> 

Think:

- CPU throttling
- Low CPU limit

Quick Check Command
```bash
kubectl top pods --sort-by=cpu
```

# 🧠 CKA Cheat Sheet

```text
CPU request
→ Scheduling

CPU limit
→ Runtime ceiling

CPU > limit
→ Throttling

Memory > limit
→ OOMKilled may occur

kubectl top pod
→ Observe CPU usage

kubectl top nodes
→ Observe Node-level usage
```

### One-line memory rule

> **CPU limit slows the container down; memory limit can kill the container.**
