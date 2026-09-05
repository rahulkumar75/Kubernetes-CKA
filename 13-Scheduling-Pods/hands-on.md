Absolutely. Add this **hands-on workflow** to Task 13. It covers the concepts in a logical order without making the document too large.

# 🧪 Hands-on Implementation — Scheduler, Static Pods, Labels & Selectors

> **Goal:** Create Pods, observe Scheduler behavior, manually assign a Pod to a Node, and practice Labels/Selectors.

---

## Step 1 — Check Cluster & Nodes

```bash
kubectl get nodes
kubectl get pods -A
kubectl config current-context
```

Identify your worker Nodes:

```bash
kubectl get nodes -o wide
```

---

## Step 2 — Observe Normal Scheduling

Create a Pod **without specifying `nodeName`**:

```bash
kubectl run nginx --image=nginx
```

Check where it was scheduled:

```bash
kubectl get pod nginx -o wide
```

Expected:

```text
NAME    READY   STATUS    NODE
nginx   1/1     Running   worker-1
```

> **Observation:** The Scheduler selected the Node.

---

## Step 3 — Manually Schedule a Pod

First, get the Node names:

```bash
kubectl get nodes
```

Create `manual-pod.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: manual-nginx
spec:
  nodeName: worker-2
  containers:
    - name: nginx
      image: nginx
```

Apply:

```bash
kubectl apply -f manual-pod.yaml
```

Verify:

```bash
kubectl get pod manual-nginx -o wide
```

The Pod should run on:

```text
worker-2
```

### 🧠 Observe

```text
No nodeName
    ↓
Scheduler chooses Node

nodeName: worker-2
    ↓
Pod goes directly to worker-2
```

---

## Step 4 — Practice Labels

Create a Pod with labels:

```bash
kubectl run web-1 \
  --image=nginx \
  --labels="app=web,env=dev"
```

Check:

```bash
kubectl get pods --show-labels
```

Filter:

```bash
kubectl get pods -l app=web
kubectl get pods -l env=dev
```

Add another label:

```bash
kubectl label pod web-1 tier=frontend
```

Verify:

```bash
kubectl get pod web-1 --show-labels
```

---

## Step 5 — Practice Selectors

Create two Pods:

```bash
kubectl run web-2 --image=nginx --labels="app=web"
kubectl run db-1 --image=nginx --labels="app=db"
```

Now select only web Pods:

```bash
kubectl get pods -l app=web
```

Expected:

```text
web-1
web-2
```

Select database Pods:

```bash
kubectl get pods -l app=db
```

### 🧠 Observe

```text
Labels
  ↓
app=web

Selector
  ↓
-l app=web

  ↓

web-1
web-2
```

---

## Step 6 — Practice Annotations

Add an annotation:

```bash
kubectl annotate pod web-1 description="Frontend application"
```

Verify:

```bash
kubectl describe pod web-1
```

Or:

```bash
kubectl get pod web-1 -o yaml
```

You will find:

```yaml
annotations:
  description: Frontend application
```

> **Remember:** Annotations provide information; they are not used to select Pods.

---

## Step 7 — Observe a Static Pod

On a kubeadm control-plane Node, check:

```bash
ls /etc/kubernetes/manifests/
```

You may see:

```text
etcd.yaml
kube-apiserver.yaml
kube-controller-manager.yaml
kube-scheduler.yaml
```

Check the corresponding Pods:

```bash
kubectl get pods -n kube-system
```

### 🧠 Observe

```text
Static Pod manifest
       ↓
/etc/kubernetes/manifests/
       ↓
Kubelet watches directory
       ↓
Kubelet runs Static Pod
```

> On environments such as **Kind**, the control-plane setup may differ from a typical kubeadm installation, so the `/etc/kubernetes/manifests/` workflow may not be directly visible from your host.

---

## Step 8 — Cleanup

```bash
kubectl delete pod nginx manual-nginx web-1 web-2 db-1
```

Verify:

```bash
kubectl get pods
```

---

# 🔥 Hands-on Recall

```text
Scheduler
→ Decides where an unscheduled Pod runs

nodeName
→ Manually specifies the Node

Label
→ Tags/identifies a resource

Selector
→ Finds resources using labels

Annotation
→ Stores additional metadata

Static Pod
→ Managed directly by Kubelet
```

### Final Verification

You should be able to answer:

1. **Who decides where a normal Pod runs?** → Scheduler
2. **How can I manually choose a Node?** → `spec.nodeName`
3. **How do I find Pods with `app=web`?** → `kubectl get pods -l app=web`
4. **Are annotations used by selectors?** → No
5. **Who manages Static Pods?** → Kubelet
