# CKA Quick Revision

### Create Pod

```bash
kubectl run nginx --image=nginx
```

### Generate YAML

```bash
kubectl run nginx --image=nginx --dry-run=client -o yaml
```

### Save YAML

```bash
kubectl run nginx --image=nginx --dry-run=client -o yaml > pod.yaml
```

### Apply YAML

```bash
kubectl apply -f pod.yaml
```

### Inspect

```bash
kubectl get pods
kubectl describe pod <pod>
kubectl get pod <pod> -o yaml
kubectl logs <pod>
```

### Labels

```bash
kubectl get pods --show-labels
kubectl get pods -l env=demo
```

### Delete

```bash
kubectl delete pod <pod>
```

### Key troubleshooting rule

**`kubectl get` → `kubectl describe` → check Events → inspect YAML/logs → fix/recreate → verify.**
