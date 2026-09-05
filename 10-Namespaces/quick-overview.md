# Quick Revision

### Create

```bash
kubectl create ns demo
```

### List

```bash
kubectl get ns
```

### Use namespace

```bash
kubectl config set-context --current --namespace=demo
```

### Check namespace

```bash
kubectl config view --minify | grep namespace
```

### Resource in namespace

```bash
kubectl get pods -n demo
```

### Deploy

```bash
kubectl create deployment nginx-demo \
  --image=nginx \
  -n demo
```

### Scale

```bash
kubectl scale deployment nginx-demo \
  --replicas=3 \
  -n demo
```

### Service

```bash
kubectl expose deployment nginx-demo \
  --name=svc-demo \
  --port=80 \
  --target-port=80 \
  -n demo
```

### DNS

```text
<service>.<namespace>.svc.cluster.local
```

---

# 🧠 Final Memory

```text
Namespace
   ↓
Logical isolation

Service
   ↓
Stable network endpoint

Same namespace
   ↓
service-name

Different namespace
   ↓
service-name.namespace

Full DNS
   ↓
service-name.namespace.svc.cluster.local
```
---