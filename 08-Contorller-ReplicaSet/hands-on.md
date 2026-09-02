# Hands-On Implementation

### Step 1 — Create a ReplicaSet

```bash
kubectl apply -f rs.yaml
```

### Step 2 — Verify

```bash
kubectl get rs
kubectl get pods --show-labels
```

### Step 3 — Scale

```bash
kubectl scale rs nginx-rs --replicas=5
```

### Step 4 — Create Deployment

```bash
kubectl create deployment nginx --image=nginx
```

### Step 5 — Scale Deployment

```bash
kubectl scale deployment nginx --replicas=3
```

### Step 6 — Update Image

```bash
kubectl set image deployment/nginx nginx=nginx:1.24
```

### Step 7 — Monitor Rollout

```bash
kubectl rollout status deployment/nginx
```

### Step 8 — Check ReplicaSets

```bash
kubectl get rs
```

### Step 9 — View History

```bash
kubectl rollout history deployment/nginx
```

### Step 10 — Rollback

```bash
kubectl rollout undo deployment/nginx
```

---