Real-time way to understand **three different Node-level pressures**:

### 1. Memory Pressure — Node is running out of RAM

Imagine a production Node:

```text
Node memory:
16 GiB total
15.5 GiB currently used
        ↓
Available memory becomes very low
        ↓
Node condition:
MemoryPressure=True
```

Even if every Pod has requests/limits, the **Node itself can still enter memory pressure**.

Example:

```text
Node
├── Pod A → limit 2Gi
├── Pod B → limit 4Gi
├── Pod C → limit 3Gi
├── Pod D → limit 2Gi
└── System processes → 5Gi
                         ↓
                   Memory pressure
```

Kubernetes may start **evicting Pods** to protect the Node.

### Real production scenario

A Java application has:

```yaml
resources:
  requests:
    memory: "1Gi"
  limits:
    memory: "4Gi"
```

The container cannot normally consume more than its 4Gi limit, but suppose many Pods are running on the same Node and the Node also needs memory for:

* kubelet
* container runtime
* OS processes
* Kubernetes system Pods

The Node can still run into memory pressure.

```text
Node memory pressure
        ↓
Kubelet detects pressure
        ↓
Pod eviction may occur
        ↓
Pod gets rescheduled elsewhere
```

**CKA command:**

```bash
kubectl describe node <node-name>
```

Look for:

```text
Conditions:
  MemoryPressure   True
```

---

## 2. Disk Pressure — Node is running out of disk

This is a very common production problem.

Imagine:

```text
Node disk = 100 GB

Container logs       → 30 GB
Container images     → 25 GB
Pod temporary files  → 20 GB
Application data     → 20 GB
Free space           → 5 GB
```

The Node can enter:

```text
DiskPressure=True
```

Kubernetes monitors resources such as:

* node filesystem
* image filesystem
* container logs
* temporary storage

### Real production scenario

An application suddenly starts generating huge logs:

```text
Application
    ↓
大量 logs
    ↓
Container log files grow
    ↓
Node disk fills
    ↓
DiskPressure=True
    ↓
Kubernetes may evict Pods
```

This is why **container logging, image cleanup, and ephemeral storage** are important in production.

Check:

```bash
kubectl describe node <node-name>
```

Look for:

```text
Conditions:
  DiskPressure   True
```

Also:

```bash
kubectl get nodes
```

You may see the Node still listed as `Ready`, but `describe node` reveals:

```text
DiskPressure=True
```

---

## 3. CPU Pressure — Slightly Different

CPU does **not have a Node condition called `CPUPressure`** like `MemoryPressure` or `DiskPressure`.

This distinction is important for CKA.

Suppose:

```text
Node:
8 CPU cores

Pod A → using 3 CPU
Pod B → using 2 CPU
Pod C → using 2 CPU
Pod D → using 1 CPU
             ↓
          8 CPU used
```

Now another workload wants CPU.

The Scheduler looks at **CPU requests** when deciding whether the Pod can be scheduled.

If there isn't enough allocatable CPU for the new Pod's request:

```text
New Pod
  ↓
CPU request = 2 CPU
  ↓
No suitable Node
  ↓
Pod remains Pending
  ↓
Events:
Insufficient cpu
```

For an already-running container, if it reaches its CPU limit:

```text
CPU usage > CPU limit
        ↓
CPU throttling
        ↓
Container gets less CPU time
```

So don't think of CPU the same way as memory/disk pressure.

---

# 🧠 CKA Memory Trick

| Node problem               | What you may see                                          |
| -------------------------- | --------------------------------------------------------- |
| **Memory pressure**        | `MemoryPressure=True`, Pod eviction/OOM-related behavior  |
| **Disk pressure**          | `DiskPressure=True`, Pod eviction                         |
| **CPU contention**         | Scheduling failure (`Insufficient cpu`) or CPU throttling |
| **CPU pressure condition** | ❌ No standard `CPUPressure=True` Node condition           |

### Final mental model

```text
                 NODE
                  │
       ┌──────────┼──────────┐
       ↓          ↓          ↓
    Memory       Disk       CPU
   Pressure    Pressure   Contention
       │          │          │
       ↓          ↓          ↓
    Eviction    Eviction   Pending /
    / OOM       possible   Throttling
```

**Most important note:**

> **Requests and limits protect individual containers and help the Scheduler make placement decisions, but they do not guarantee that the Node itself will never experience resource pressure.**
