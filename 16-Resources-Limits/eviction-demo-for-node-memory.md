## 🧪 Real Node Memory Eviction Demo

The important distinction is:

> **OOMKilled** = container exceeded its memory limit.
> **Eviction** = kubelet removes Pods from a Node because the Node is under resource pressure.


### 🎯 Objective

* Create real **Node MemoryPressure**.
* Observe how kubelet reacts to memory pressure.
* Understand **Pod eviction**.
* See why Requests/Limits don't completely prevent Node-level pressure.
* Practice CKA troubleshooting commands.

> ⚠️ **Run this only on a disposable kind/minikube/lab cluster.** Do not perform uncontrolled memory stress on a production Node.

---

# 1. Check the Node

```bash
kubectl get nodes
```

Then:

```bash
kubectl describe node <node-name>
```

Look at:

```text
Conditions:
  MemoryPressure   False
  DiskPressure     False
  Ready            True
```

We want to observe:

```text
MemoryPressure: False → True
```

---

# 2. Check Current Memory Usage

If Metrics Server is installed:

```bash
kubectl top nodes
```

Also:

```bash
kubectl top pods -A
```

This gives you a baseline before starting the experiment.

---

# 3. Create a Test Namespace

```bash
kubectl create namespace eviction-demo
```

---

# 4. Create a Memory Stress Pod

Create `memory-stress.yaml`:

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: memory-stress
  namespace: eviction-demo

spec:
  containers:
    - name: stress
      image: polinux/stress

      resources:
        requests:
          memory: "100Mi"
        limits:
          memory: "4Gi"

      command: ["stress"]
      args:
        - "--vm"
        - "1"
        - "--vm-bytes"
        - "3G"
        - "--vm-hang"
        - "0"
```

Apply:

```bash
kubectl apply -f memory-stress.yaml
```

Check:

```bash
kubectl get pod -n eviction-demo -o wide
```

---

# 5. Observe the Node

Run:

```bash
kubectl top node
```

You should see memory consumption increasing.

At the same time, in another terminal:

```bash
watch kubectl describe node <node-name>
```

Look for:

```text
Conditions:
  MemoryPressure
```

Initially:

```text
MemoryPressure   False
```

Under sufficient pressure, it may become:

```text
MemoryPressure   True
```

---

# 6. Observe Pod Eviction

Check:

```bash
kubectl get pods -n eviction-demo -w
```

You may eventually observe the Pod being:

```text
Evicted
```

or a Pod termination followed by rescheduling/recreation if it is managed by a controller.

Check:

```bash
kubectl get pods -n eviction-demo
```

Then:

```bash
kubectl describe pod memory-stress -n eviction-demo
```

Look for information related to eviction and Node resource pressure.

---

# 7. Understand What Actually Happened

The important flow is:

```text
memory-stress
      ↓
Consumes large amount of memory
      ↓
Node available memory becomes low
      ↓
Kubelet detects memory pressure
      ↓
MemoryPressure=True
      ↓
Kubelet starts eviction
      ↓
Pods may be evicted
```

This is **different from OOMKilled**.

### OOMKilled

```text
Container
   ↓
Exceeds its memory limit
   ↓
Container killed
   ↓
OOMKilled
```

### Node eviction

```text
Node
   ↓
Available memory becomes dangerously low
   ↓
MemoryPressure
   ↓
Kubelet evicts Pods
```

---

# 8. Why Didn't the 4Gi Limit Protect the Node?

This is the important concept from your Requests & Limits notes.

We configured:

```yaml
limits:
  memory: "4Gi"
```

That limits the **container**.

It does **not** mean:

> "The Node can never run out of memory."

The Node also needs memory for:

```text
Linux OS
kubelet
container runtime
Kubernetes system components
other Pods
container processes
```

Therefore:

```text
Pod limits
     ↓
Protect individual containers
     ↓
BUT
     ↓
Node can still experience memory pressure
```

---

# 9. See Which Pods Are Evicted

After the experiment:

```bash
kubectl get pods -A
```

You can also check recent events:

```bash
kubectl get events -A --sort-by=.lastTimestamp
```

Look for messages related to:

```text
Evicted
MemoryPressure
```

For Node-specific information:

```bash
kubectl describe node <node-name>
```

Pay particular attention to:

```text
Conditions
Events
Allocated resources
```

---

# 10. A Better Realistic Demo with a Deployment

A Deployment makes the behavior easier to understand because Kubernetes can recreate the Pod.

Create:

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: memory-app
  namespace: eviction-demo

spec:
  replicas: 3

  selector:
    matchLabels:
      app: memory-app

  template:
    metadata:
      labels:
        app: memory-app

    spec:
      containers:
        - name: stress
          image: polinux/stress

          resources:
            requests:
              memory: "100Mi"
            limits:
              memory: "2Gi"

          command: ["stress"]
          args:
            - "--vm"
            - "1"
            - "--vm-bytes"
            - "1500M"
            - "--vm-hang"
            - "0"
```

Apply:

```bash
kubectl apply -f memory-deployment.yaml
```

Check:

```bash
kubectl get pods -n eviction-demo -o wide
```

Now watch:

```bash
kubectl get pods -n eviction-demo -w
```

If the Node enters memory pressure and Pods are evicted, the **Deployment controller attempts to maintain 3 replicas**.

This demonstrates an important Kubernetes pattern:

```text
Node MemoryPressure
        ↓
Pod eviction
        ↓
Deployment sees replica count < desired
        ↓
New Pod created
        ↓
Scheduler finds another suitable Node
        ↓
Pod runs elsewhere
```

If there is **no other Node with enough resources**, the replacement Pod can remain:

```text
Pending
```

---

# 11. CKA Troubleshooting Scenario

Imagine you receive this alert:

> "Application Pod disappeared from the Node."

Your troubleshooting flow:

### Step 1 — Check Pods

```bash
kubectl get pods -A -o wide
```

### Step 2 — Check Pod details

```bash
kubectl describe pod <pod-name> -n <namespace>
```

### Step 3 — Check Node conditions

```bash
kubectl describe node <node-name>
```

Look for:

```text
MemoryPressure=True
DiskPressure=True
```

### Step 4 — Check events

```bash
kubectl get events -A --sort-by=.lastTimestamp
```

### Step 5 — Check resource usage

```bash
kubectl top nodes
kubectl top pods -A --sort-by=memory
```

---

# 🧠 CKA: OOMKilled vs Evicted

| Scenario                           | Cause                    | Result               |
| ---------------------------------- | ------------------------ | -------------------- |
| Container exceeds memory limit     | Container-level limit    | `OOMKilled`          |
| Node runs critically low on memory | Node-level pressure      | Pod may be `Evicted` |
| Node runs low on disk              | Node-level disk pressure | Pod may be `Evicted` |
| CPU reaches container limit        | CPU limit                | Throttling           |

### One-line memory rule

> **OOMKilled = container problem. Evicted = Node pressure problem.**

---

# 🧹 Cleanup

Delete the entire lab:

```bash
kubectl delete namespace eviction-demo
```

---

### ⚠️ Important lab note

A memory-eviction demo is **not guaranteed to trigger at exactly the memory amount you specify**. Kubelet eviction depends on the Node's available memory, eviction thresholds, system reservations, workload placement, and other factors.

For a CKA lab, the key thing to learn is the **diagnostic chain**:

```text
Pod problem
   ↓
describe pod
   ↓
Was it OOMKilled?

OR

Node problem
   ↓
describe node
   ↓
MemoryPressure / DiskPressure?
   ↓
Events
   ↓
Eviction
```

This distinction is much more valuable for the exam than simply forcing a Pod into `Evicted`.
