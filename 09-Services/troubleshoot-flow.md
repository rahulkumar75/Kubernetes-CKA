# Quick Revision

### Service

> **Stable network endpoint for accessing Pods.**

### ClusterIP

```text
Internal access
```

### NodePort

```text
NodeIP:30000–32767
```

### LoadBalancer

```text
External cloud load balancer
```

### ExternalName

```text
Service name → External DNS name
```

### Most Important Commands

```bash
kubectl get svc

kubectl describe svc <service>

kubectl get pods --show-labels

kubectl get endpoints <service>

kubectl get endpointslices

kubectl get svc -o wide
```

### Troubleshooting Memory

```text
Service not working
       ↓
Check Service
       ↓
Check Selector
       ↓
Check Pod Labels
       ↓
Check Endpoints
       ↓
Check Pod Status / Events
```

> **CKA Rule:** If a Service has **no endpoints**, first check the **Service selector ↔ Pod labels**.
