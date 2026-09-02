# Troubleshooting Flow

If the expected number of Pods is not running:

```text
kubectl get deployment
        ↓
kubectl get rs
        ↓
kubectl get pods --show-labels
        ↓
kubectl describe rs <name>
        ↓
Check selector ↔ Pod labels
        ↓
kubectl describe pod <pod>
        ↓
Check Events
```

### Most important checks

```bash
kubectl get pods --show-labels
kubectl get rs
kubectl describe rs <name>
kubectl describe pod <pod-name>
```

---

# Quick Memory

```text
RC
└── Legacy controller

ReplicaSet
└── Maintains Pod replicas

Deployment
└── Manages ReplicaSets
    ├── Scaling
    ├── Rolling updates
    ├── Rollback
    └── Revision history
```

### Remember

> **Labels → identify Pods**
> **Selectors → select Pods**
> **ReplicaSet → maintain replicas**
> **Deployment → manage application rollouts**

### Most-used commands

```bash
kubectl get pods --show-labels

kubectl scale rs <name> --replicas=5

kubectl scale deployment <name> --replicas=5

kubectl set image deployment/<name> <container>=<image>

kubectl rollout status deployment/<name>

kubectl rollout history deployment/<name>

kubectl rollout undo deployment/<name>
```

