# Hands-On Tasks

## Task 1 — Create Pod Imperatively

Create an Nginx Pod:

```bash
kubectl run nginx-imp --image=nginx:latest
```

Verify:

```bash
kubectl get pods
kubectl get pod nginx-imp -o wide
```

---

## Task 2 — Generate YAML

Generate YAML without creating the Pod:

```bash
kubectl run nginx-new \
  --image=nginx \
  --dry-run=client \
  -o yaml > new-pod.yaml
```

Inspect:

```bash
cat new-pod.yaml
```

Apply:

```bash
kubectl apply -f new-pod.yaml
```

Verify:

```bash
kubectl get pod nginx-new
```

Check labels:

```bash
kubectl get pod nginx-new --show-labels
```

---

## Task 3 — Troubleshoot a Broken Pod

Create a Pod using an incorrect image:

```bash
kubectl run nginx-error --image=nginx123
```

### Step 1 — Check status

```bash
kubectl get pods
```

Expected:

```text
nginx-error   0/1   ImagePullBackOff
```

### Step 2 — Describe the Pod

```bash
kubectl describe pod nginx-error
```

Look at:

```text
Events:
```

Identify the image-pull error.

### Step 3 — Confirm the image

```bash
kubectl get pod nginx-error -o yaml
```

Find:

```yaml
image: nginx123
```

### Step 4 — Fix

Because the Pod specification/image is not something you should rely on editing in place, recreate it:

```bash
kubectl delete pod nginx-error

kubectl run nginx-error --image=nginx
```

### Step 5 — Verify

```bash
kubectl get pods
```

Expected:

```text
nginx-error   1/1   Running
```

---