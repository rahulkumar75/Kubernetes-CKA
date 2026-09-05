# 🚨 CKA Debugging: 

 A high-value troubleshooting scenario because NoExecute behaves differently from NoSchedule.

## Pod Eviction with `NoExecute`

## 🧠 The Trap

Remember:

| Effect             | New Pod    | Existing Pod    |
| ------------------ | ---------- | --------------- |
| `NoSchedule`       | ❌ Blocked  | ✅ Keeps running |
| `PreferNoSchedule` | ⚠️ Avoided | ✅ Keeps running |
| `NoExecute`        | ❌ Blocked  | ❌ **Evicted**   |

> **CKA shortcut:** `NoExecute` = **"Don't schedule + remove existing Pods."**

---

# 🧪 Hands-on Scenario

### Step 1 — Check Nodes

```bash
kubectl get nodes
```

Assume:

```text
worker-1
worker-2
```

---

### Step 2 — Create a Pod

```bash
kubectl run nginx --image=nginx
```

Check placement:

```bash
kubectl get pod nginx -o wide
```

Example:

```text
NAME    STATUS    NODE
nginx   Running   worker-1
```

---

## Step 3 — Apply `NoExecute`

Apply a taint to the Node where the Pod is running:

```bash
kubectl taint node worker-1 maintenance=true:NoExecute
```

Immediately check:

```bash
kubectl get pods -o wide
```

### 🔥 What happened?

The existing `nginx` Pod is **evicted** because it doesn't tolerate:

```text
maintenance=true:NoExecute
```

The important part is that `NoExecute` affects **already-running Pods**.

---

# 🔍 Step 4 — Check the Pod

```bash
kubectl describe pod nginx
```

You may see termination/eviction-related information.

Also check:

```bash
kubectl get events --sort-by=.lastTimestamp
```

This is an important CKA debugging command.

---

# 🧪 Step 5 — Create a Pod with Toleration

Create:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-tolerated
spec:
  tolerations:
    - key: maintenance
      operator: Equal
      value: "true"
      effect: NoExecute
  containers:
    - name: nginx
      image: nginx
```

Apply:

```bash
kubectl apply -f nginx-tolerated.yaml
```

Check:

```bash
kubectl get pod nginx-tolerated -o wide
```

This Pod **can run on the tainted Node**.

---

# ⚠️ CKA Trap: `tolerationSeconds`

A Pod can tolerate `NoExecute` **temporarily**.

Example:

```yaml
tolerations:
  - key: maintenance
    operator: Equal
    value: "true"
    effect: NoExecute
    tolerationSeconds: 30
```

Meaning:

```text
NoExecute applied
       ↓
Pod continues running
       ↓
30 seconds
       ↓
Pod is evicted
```

This is useful when you want a Pod to tolerate a temporary Node condition.

---

# 🔥 Real CKA Debugging Question

### Scenario

You run:

```bash
kubectl get pods -o wide
```

and notice:

```text
nginx    0/1    Terminating
```

or the Pod disappears/restarts after a Node taint is applied.

You check:

```bash
kubectl describe node worker-1
```

and find:

```text
Taints:
  maintenance=true:NoExecute
```

### What do you check next?

```bash
kubectl get pod nginx -o yaml
```

Look for:

```yaml
spec:
  tolerations:
```

If there is **no matching `NoExecute` toleration**, the Pod can be evicted.

---

# 🧠 CKA Debugging Flow

Memorize this:

```text
Pod Pending?
     ↓
Check Node
     ↓
kubectl describe node <node>
     ↓
Check Taints
     ↓
NoSchedule?
     ↓
Check Pod tolerations
```

For eviction:

```text
Pod was Running
     ↓
Suddenly Evicted/Terminated
     ↓
Check Node taints
     ↓
Found NoExecute
     ↓
Check Pod toleration
     ↓
No matching toleration?
     ↓
Pod gets evicted
```

### ⭐ Final Recall

> **`NoSchedule` → prevents new Pods.**
> **`NoExecute` → prevents new Pods + evicts existing Pods.**
> **`tolerationSeconds` → temporary protection from `NoExecute`.**
