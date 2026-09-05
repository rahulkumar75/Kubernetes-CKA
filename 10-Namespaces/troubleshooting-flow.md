# Troubleshooting Namespace Connectivity

If communication does not work, check:

### 1. Service

```bash
kubectl get svc -n demo
```

### 2. Service selector and details

```bash
kubectl describe svc svc-demo -n demo
```

### 3. Endpoints

```bash
kubectl get endpoints svc-demo -n demo
```

If there are no endpoints, check:

```bash
kubectl get pods -n demo --show-labels
```

The Service selector must match the Pod labels.

### 4. Pod status

```bash
kubectl get pods -n demo
kubectl describe pod <pod-name> -n demo
```

---
