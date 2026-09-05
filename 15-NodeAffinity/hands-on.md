Practice for **CKA (real exam style)** 🚀

1. ✅ **Real exam-style YAML questions (with solutions)**
2. ✅ **Hands-on lab (step-by-step commands)**

---

# ✅ PART 1 — Real CKA Exam YAML Questions

---

## 🧪 Question 1 — Strict Node Affinity

👉 **Task:**

Create a Pod named `nginx-pod` that:

- Uses image `nginx`
- Runs **only on nodes with label** `env=prod`

---

### ✅ Solution YAML

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
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

## 🧪 Question 2 — Preferred Node Affinity

👉 **Task:**

Create a Pod:

- Name: `soft-pod`
- Prefer nodes with `disk=ssd`
- But allow scheduling anywhere

---

### ✅ Solution YAML

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: soft-pod
spec:
  containers:
  - name: nginx
    image: nginx
  affinity:
    nodeAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 1
        preference:
          matchExpressions:
          - key: disk
            operator: In
            values:
            - ssd
```

---

## 🧪 Question 3 — Node Affinity + Toleration

👉 **Task:**

- Pod name: `secure-pod`
- Should run only on nodes:
    - Label: `secure=true`
    - Taint: `secure=true:NoSchedule`

---

### ✅ Solution YAML

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secure-pod
spec:
  containers:
  - name: nginx
    image: nginx

  tolerations:
  - key: "secure"
    operator: "Equal"
    value: "true"
    effect: "NoSchedule"

  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: secure
            operator: In
            values:
            - "true"
```

---

## 🧪 Question 4 — Using Exists Operator

👉 **Task:**

Schedule Pod only on nodes where label `gpu` exists

---

### ✅ Solution YAML

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: gpu-pod
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

## 🧪 Question 5 — Multiple Conditions

👉 **Task:**

Run Pod only if:

- `env=prod`
- AND `disk=ssd`

---

### ✅ Solution YAML

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multi-condition-pod
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
            values: ["prod"]
          - key: disk
            operator: In
            values: ["ssd"]
```

---

# ✅ PART 2 — Hands-on Lab (Step-by-Step)

👉 This is how you should practice locally (VERY IMPORTANT for CKA)

---

## 🔧 Step 1 — Check Nodes

```bash
kubectl get nodes
```

---

## 🔧 Step 2 — Add Labels to Nodes

👉 Pick a node name from above

```bash
kubectl label node <node-name> env=prod
kubectl label node <node-name> disk=ssd
kubectl label node <node-name> secure=true
```

✔ Verify:

```bash
kubectl get nodes --show-labels
```

---

## 🔧 Step 3 — Add Taint to Node

```bash
kubectl taint node <node-name> secure=true:NoSchedule
```

✔ Verify:

```bash
kubectl describe node <node-name> | grep Taint
```

---

## 🔧 Step 4 — Apply Pod YAML

```bash
kubectl apply -f pod.yaml
```

---

## 🔧 Step 5 — Check Scheduling

```bash
kubectl get pods -o wide
```

👉 Check:

- Node assigned or not

---

## 🔧 Step 6 — Debug (VERY IMPORTANT for exam)

If Pod not scheduled:

```bash
kubectl describe pod <pod-name>
```

👉 Look for:

- `node affinity mismatch`
- `taint issue`

---

## 🔧 Step 7 — Test Label Removal

```bash
kubectl label node <node-name> env-
```

👉 Then create new pod → observe behavior

---

## 🔧 Step 8 — Cleanup

```bash
kubectl delete pod <pod-name>
kubectl taint node <node-name> secure=true:NoSchedule-
kubectl label node <node-name> env-
kubectl label node <node-name> disk-
kubectl label node <node-name> secure-
```

---

# 🎯 Pro CKA Tips (Important)

👉 In exam:

- Don’t write YAML from scratch → use:

```bash
kubectl run nginx --image=nginx --dry-run=client -o yaml > pod.yaml
```

👉 Then edit → add affinity

---

👉 Always debug with:

```bash
kubectl describe pod
```

---

👉 Fast check:

```bash
kubectl get pod -o wide
```

---
