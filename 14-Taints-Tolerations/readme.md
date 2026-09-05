# **Taints and Tolerations in Kubernetes 🚧**

## 🎯 Objective

By the end of this task, you should be able to:

* Understand **Taints, Tolerations, and their effects**.
* Control which Pods can be scheduled on specific Nodes.
* Use **labels + `nodeSelector`** for basic node selection.
* Troubleshoot Pods stuck in `Pending` due to scheduling constraints.

---

# 1. Taints

A **Taint is applied to a Node** to prevent Pods from being scheduled there unless they have a matching toleration.

Think:

> **Taint = Node says: "Don't schedule here unless you are allowed."**

Example:

```bash
kubectl taint nodes node1 gpu=true:NoSchedule
```

This means:

```text
Node
  ↓
gpu=true:NoSchedule
  ↓
Pod must have matching toleration
```

### Remove a Taint

```bash
kubectl taint nodes node1 gpu=true:NoSchedule-
```

---

# 2. Tolerations

A **Toleration is defined on a Pod** and allows it to be scheduled onto a Node with a matching taint.

```yaml
tolerations:
  - key: "gpu"
    operator: "Equal"
    value: "true"
    effect: "NoSchedule"
```

> ⚠️ **Important:** A toleration allows a Pod to be considered for the Node; it does **not guarantee** that the Pod will be scheduled there.

To guarantee/select a particular Node, combine tolerations with **nodeSelector or node affinity**.

---

# 3. Taint Effects

| Effect             | Meaning                                                             |
| ------------------ | ------------------------------------------------------------------- |
| `NoSchedule`       | New Pods without a matching toleration won't be scheduled           |
| `PreferNoSchedule` | Scheduler tries to avoid the Node                                   |
| `NoExecute`        | Existing non-tolerating Pods are evicted; new ones are also blocked |

---

# 4. Labels vs Taints

| Concept        | Applied to    | Purpose                       |
| -------------- | ------------- | ----------------------------- |
| **Label**      | Node/Pod/etc. | Identify and select resources |
| **Taint**      | Node          | Repel unwanted Pods           |
| **Toleration** | Pod           | Allow Pod onto a tainted Node |

### 🧠 Easy memory

> **Taint = Node says NO**
> **Toleration = Pod says I CAN**
> **NodeSelector/Affinity = Pod says I WANT**

---

# 🧪 Hands-on Implementation

## Step 1 — Check Nodes

```bash
kubectl get nodes
kubectl get nodes --show-labels
```

Choose one worker Node for the experiment.

Example:

```text
cka-worker
```

---

## Step 2 — Apply a Taint

```bash
kubectl taint node cka-worker gpu=true:NoSchedule
```

Verify:

```bash
kubectl describe node cka-worker | grep -i taint
```

Expected:

```text
Taints: gpu=true:NoSchedule
```

---

## Step 3 — Create a Pod Without Toleration

```bash
kubectl run nginx-test --image=nginx
```

Check:

```bash
kubectl get pod nginx-test -o wide
```

The Pod should remain:

```text
Pending
```

Troubleshoot:

```bash
kubectl describe pod nginx-test
```

Look at the **Events** section. You should see that the Node cannot be used because of the taint.

> If other untainted Nodes are available, the Pod may run on another Node. To reproduce `Pending`, make sure no suitable Node is available.

---

## Step 4 — Add a Toleration

Create `toleration-pod.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-toleration
spec:
  tolerations:
    - key: gpu
      operator: Equal
      value: "true"
      effect: NoSchedule
  containers:
    - name: nginx
      image: nginx
```

Apply:

```bash
kubectl apply -f toleration-pod.yaml
```

Verify:

```bash
kubectl get pod nginx-toleration -o wide
```

The Pod **can now be scheduled** on the tainted Node.

> It may still be scheduled on another suitable Node. Toleration alone does not force placement.

---

# 5. Taint + Label + nodeSelector

If the requirement is:

> **"Only GPU Pods should run on the GPU Node."**

Use both:

```text
Taint
  ↓
Prevents unwanted Pods

Toleration
  ↓
Allows GPU Pods

nodeSelector / Affinity
  ↓
Selects GPU Node
```

### Label the Node

```bash
kubectl label node cka-worker hardware=gpu
```

Verify:

```bash
kubectl get nodes --show-labels
```

---

## Pod YAML

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: gpu-nginx
spec:
  nodeSelector:
    hardware: gpu

  tolerations:
    - key: gpu
      operator: Equal
      value: "true"
      effect: NoSchedule

  containers:
    - name: nginx
      image: nginx
```

Apply:

```bash
kubectl apply -f gpu-nginx.yaml
```

Verify placement:

```bash
kubectl get pod gpu-nginx -o wide
```

Expected:

```text
NODE
cka-worker
```

### 🧠 Why both?

* **Taint/Toleration:** "Can this Pod use the Node?"
* **nodeSelector:** "Which Node does the Pod want?"

---

# 6. Remove the Taint

```bash
kubectl taint node cka-worker gpu=true:NoSchedule-
```

Verify:

```bash
kubectl describe node cka-worker | grep -i taint
```

Now a Pod without the toleration can also be scheduled there.

---

# 🔧 Troubleshooting

If a Pod remains `Pending`:

```bash
kubectl get pods -o wide
kubectl describe pod <pod-name>
kubectl describe node <node-name>
```

Check for:

* Untolerated taints
* Incorrect `nodeSelector`
* Missing/incorrect node labels
* Insufficient CPU/memory
* Node not `Ready`

### Common CKA clue

If you see:

```text
untolerated taint
```

Think:

> **Check the Node's taints and the Pod's tolerations.**

---

# 🧠 Final Recall

```text
Taint
→ Applied to Node
→ Repels Pods

Toleration
→ Applied to Pod
→ Allows the Pod onto the tainted Node

nodeSelector
→ Selects Node using labels

Node Affinity
→ Advanced Node selection rules
```

> **Taints repel. Tolerations permit. Labels identify. Selectors/Affinity choose.**
