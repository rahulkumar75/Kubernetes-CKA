# Affinity & Anti-Affinity (Kubernetes)

👉 These control **how Pods are placed relative to other Pods or Nodes**

---

# Big Picture

| Type | Works On | Purpose |
| --- | --- | --- |
| Node Affinity | Nodes | Choose node based on labels |
| Pod Affinity | Pods | Place pod **near other pods** |
| Pod Anti-Affinity | Pods | Keep pod **away from other pods** |

---

# Concept Visualization

![alt text](image-5.png)

---

# 🔥 1. Pod Affinity (Attraction)

👉 “Schedule this pod **close to another pod**”

---

## ✅ Example Use Case

- App + Cache (Redis)
- Frontend + Backend
- Microservices needing **low latency**

---

## 🧪 YAML Example

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-affinity
spec:
  containers:
  - name: nginx
    image: nginx

  affinity:
    podAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
          - key: app
            operator: In
            values:
            - backend
        topologyKey: "kubernetes.io/hostname"
```

---

## 🧠 Meaning

- Find pods with label `app=backend`
- Schedule this pod on **same node**

👉 `topologyKey = kubernetes.io/hostname`

= same node

---

# ❄️ 2. Pod Anti-Affinity (Repulsion)

👉 “Do NOT schedule this pod near another pod”

---

## ✅ Use Case

- High availability
- Avoid single point of failure

---

## 🧪 YAML Example

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-anti-affinity
spec:
  containers:
  - name: nginx
    image: nginx

  affinity:
    podAntiAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
          - key: app
            operator: In
            values:
            - nginx
        topologyKey: "kubernetes.io/hostname"
```

---

## 🧠 Meaning

- Don’t place pods with same label on same node

👉 Ensures:

- Pods spread across nodes

---

# ⚡ Required vs Preferred (Same Concept)

| Type | Behavior |
| --- | --- |
| Required | MUST follow rule |
| Preferred | Try but not mandatory |

---

# 🎯 Key Concept: `topologyKey`

👉 Defines **scope of rule**

| Value | Meaning |
| --- | --- |
| `kubernetes.io/hostname` | Same node |
| `topology.kubernetes.io/zone` | Same zone |
| `topology.kubernetes.io/region` | Same region |

---

# 🔥 Real CKA Question

👉 “Ensure 3 replicas of nginx are NOT scheduled on same node”

---

## ✅ Solution (Deployment)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deploy
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx

      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchLabels:
                app: nginx
            topologyKey: "kubernetes.io/hostname"
```

---

# 🧪 Hands-On Lab

---

## 🔧 Step 1 — Create Backend Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: backend
  labels:
    app: backend
spec:
  containers:
  - name: nginx
    image: nginx
```

```bash
kubectl apply -f backend.yaml
```

---

## 🔧 Step 2 — Apply Pod Affinity

```bash
kubectl apply -f pod-affinity.yaml
```

✔ Check:

```bash
kubectl get pods -o wide
```

👉 Both pods should be on **same node**

---

## 🔧 Step 3 — Test Anti-Affinity

```bash
kubectl apply -f nginx-deploy.yaml
```

✔ Check:

```bash
kubectl get pods -o wide
```

👉 Pods should be on **different nodes**

---

## 🔧 Step 4 — Debug

```bash
kubectl describe pod <pod-name>
```

Look for:

- `0/3 nodes available`
- `pod anti-affinity rules not satisfied`

---

# ⚠️ Common Mistakes (VERY IMPORTANT)

❌ Forget `topologyKey` → YAML invalid

❌ Using required when cluster small → pod Pending

❌ Label mismatch → no scheduling

❌ Anti-affinity with 1 node cluster → FAIL

---

# 🧠 Final Summary (Interview Ready)

👉

- **Pod Affinity** = place pods together
- **Pod Anti-Affinity** = spread pods apart
- **TopologyKey** = defines boundary
- **Required vs Preferred** = strict vs soft

---

# 🎯 One-Line Summary

> Affinity = “stay together”
> 
> 
> Anti-Affinity = “stay apart”
> 

---