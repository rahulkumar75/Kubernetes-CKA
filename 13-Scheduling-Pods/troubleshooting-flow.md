# 🛠️ Lab Setup & Troubleshooting

## 1. Lab Setup

Use a Kubernetes cluster with **at least 2 Nodes**.

### Check cluster

```bash
kubectl config current-context
kubectl get nodes -o wide
kubectl get pods -A
```

Confirm Nodes are `Ready`:

```text
NAME       STATUS
worker-1   Ready
worker-2   Ready
```

### Identify the Scheduler

On a control-plane Node:

```bash
kubectl get pods -n kube-system | grep scheduler
```

> **Note:** The Scheduler is itself a Pod in kubeadm-based clusters, but it is typically a **Static Pod** managed by the Kubelet.

---

# 2. Troubleshooting Workflow

## 🔴 Pod Stuck in `Pending`

First:

```bash
kubectl get pod <pod-name>
kubectl describe pod <pod-name>
```

Always check the **Events** section.

Common causes:

```text
Insufficient CPU/Memory
Node selector mismatch
Node affinity mismatch
Taint without toleration
Invalid nodeName
No eligible Node
```

Check Nodes:

```bash
kubectl get nodes
kubectl describe nodes
```

---

## 🔴 Pod Not Scheduled to Expected Node

Check:

```bash
kubectl get pod <pod-name> -o wide
kubectl get pod <pod-name> -o yaml
```

Look for:

```yaml
nodeName:
nodeSelector:
affinity:
tolerations:
```

For a manually scheduled Pod:

```yaml
spec:
  nodeName: worker-2
```

Remember:

> `nodeName` bypasses normal Scheduler selection.

---

## 🔴 Label Selector Returns No Pods

Check the actual labels:

```bash
kubectl get pods --show-labels
```

Then test the selector:

```bash
kubectl get pods -l app=web
```

Compare:

```text
Pod:
app=web

Selector:
app=web
```

If the values don't match, the Pod won't be selected.

### Useful debugging command

```bash
kubectl get pods -l app=web --show-labels
```

---

## 🔴 Static Pod Not Running

On the control-plane Node:

```bash
ls /etc/kubernetes/manifests/
```

Check Kubelet:

```bash
systemctl status kubelet
```

Check recent logs:

```bash
journalctl -u kubelet -n 50
```

Check Static Pods:

```bash
kubectl get pods -n kube-system -o wide
```

> **Important:** The exact Static Pod directory and control-plane setup can vary by Kubernetes distribution. `/etc/kubernetes/manifests/` is the common kubeadm location.

---

# 🧠 CKA Troubleshooting Pattern

When a Pod is not running:

```text
kubectl get pod
       ↓
kubectl describe pod
       ↓
Check Events
       ↓
Check Node status
       ↓
Check scheduling constraints
       ↓
Fix → Verify
```

### Quick Recall

| Problem          | First command                    |
| ---------------- | -------------------------------- |
| Pod Pending      | `kubectl describe pod <pod>`     |
| Wrong Node       | `kubectl get pod <pod> -o wide`  |
| Selector issue   | `kubectl get pods --show-labels` |
| Node issue       | `kubectl describe node <node>`   |
| Static Pod issue | `systemctl status kubelet`       |
| Kubelet issue    | `journalctl -u kubelet -n 50`    |
