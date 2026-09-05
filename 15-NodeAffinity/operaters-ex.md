Operators are used in:

- Node Affinity
- Pod Affinity
- Pod Anti-Affinity
- Tolerations (`Exists`, `Equal`)

**Real YAML examples for each operator** 👇

---

# 🚀 Operators in Affinity (`matchExpressions`)

Main operators:

1. `In`
2. `NotIn`
3. `Exists`
4. `DoesNotExist`
5. `Gt`
6. `Lt`

---

# 1️⃣ `In`

👉 Node label value must match one of listed values

### ✅ Example:

Run pod only on nodes with `env=prod`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-in
spec:
  containers:
  - name: nginx
    image: nginx

  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: env
            operator: In
            values:
            - prod
```

---

# 2️⃣ `NotIn`

👉 Node label value must NOT match listed values

### ✅ Example:

Avoid nodes with `env=dev`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-notin
spec:
  containers:
  - name: nginx
    image: nginx

  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: env
            operator: NotIn
            values:
            - dev
```

---

# 3️⃣ `Exists`

👉 Label key must exist (value anything)

### ✅ Example:

Schedule only on nodes having label `gpu`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-exists
spec:
  containers:
  - name: nginx
    image: nginx

  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: gpu
            operator: Exists
```

---

# 4️⃣ `DoesNotExist`

👉 Label key must NOT exist

### ✅ Example:

Avoid nodes labeled `maintenance`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-no-label
spec:
  containers:
  - name: nginx
    image: nginx

  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: maintenance
            operator: DoesNotExist
```

---

# 5️⃣ `Gt` (Greater Than)

👉 Numeric label value greater than given number

### Example Node:

```bash
kubectl label node node1 cpu=8
```

### Pod YAML:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-gt
spec:
  containers:
  - name: nginx
    image: nginx

  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: cpu
            operator: Gt
            values:
            - "4"
```

👉 Schedules on nodes where cpu > 4

---

# 6️⃣ `Lt` (Less Than)

👉 Numeric label less than given number

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-lt
spec:
  containers:
  - name: nginx
    image: nginx

  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: cpu
            operator: Lt
            values:
            - "10"
```

👉 cpu < 10

---

# 🔥 Toleration Operators (Different Topic)

Only two commonly used:

| Operator | Meaning |
| --- | --- |
| Equal | key + value match |
| Exists | only key exists |

---

## `Equal`

```yaml
tolerations:
- key: color
  operator: Equal
  value: blue
  effect: NoSchedule
```

---

## `Exists`

```yaml
tolerations:
- key: color
  operator: Exists
  effect: NoSchedule
```

👉 tolerates any taint with key=color

---

# 🎯 Quick Memory Trick

| Operator | Meaning |
| --- | --- |
| In | Match listed values |
| NotIn | Avoid listed values |
| Exists | Key present |
| DoesNotExist | Key absent |
| Gt | Greater than |
| Lt | Less than |

---

# ⚠️ CKA Important Notes

✔ `Gt/Lt` only numeric values

✔ `Exists/DoesNotExist` no values needed

✔ `In/NotIn` values mandatory

---

# 🚀 Real Exam Practice

Generate YAML quickly:

```bash
kubectl run nginx --image=nginx --dry-run=client -o yaml > pod.yaml
```

Then add affinity block.

---

# 🎯 One-Line Summary

> Operators define **how scheduler compares labels** before placing pods.
> 

---