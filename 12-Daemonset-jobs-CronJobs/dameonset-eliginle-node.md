### DaemonSet Node Eligibility

**Eligible Node** means a Node that satisfies the DaemonSet's scheduling rules.

By default:

```text
DaemonSet → Pod on every schedulable Node
```

But a Pod may **not** run on every Node if you configure:

* `nodeSelector`
* `nodeAffinity`
* `taints/tolerations`

### Example

```yaml
spec:
  template:
    spec:
      nodeSelector:
        disktype: ssd
```

Now:

```text
Node 1 → disktype=ssd    ✅ Pod
Node 2 → disktype=hdd    ❌ No Pod
Node 3 → disktype=ssd    ✅ Pod
```

### 🧠 CKA Recall

> **DaemonSet = 1 Pod per Node that is eligible for scheduling.**

So don't memorize **"1 Pod on every Node"** as an absolute rule.
Memorize:

**`DaemonSet → 1 Pod per eligible Node`**
