# 11. Hands-On Implementation

## Step 1 — Create an Nginx Pod

```bash
kubectl run nginx --image=nginx
```

Add the required label:

```bash
kubectl label pod nginx app=nginx
```

Verify:

```bash
kubectl get pod nginx --show-labels
```

---

## Step 2 — Create ClusterIP Service

```bash
kubectl apply -f clusterip.yaml
```

Verify:

```bash
kubectl get svc
kubectl get endpoints nginx-clusterip
```

---

## Step 3 — Test Internal Access

Create a temporary Pod:

```bash
kubectl run curl --image=curlimages/curl -it --rm -- sh
```

Inside the Pod:

```bash
curl http://nginx-clusterip:80
```

This verifies:

```text
Temporary Pod
      ↓
Service DNS
      ↓
ClusterIP
      ↓
Nginx Pod
```

Exit:

```bash
exit
```

---

## Step 4 — Create NodePort

```bash
kubectl apply -f nodeport.yaml
```

Verify:

```bash
kubectl get svc nginx-nodeport
kubectl get endpoints nginx-nodeport
```

If using Kind with appropriate port mapping:

```bash
curl http://localhost:30001
```

---

## Step 5 — Test Failure Scenario

Change the Service selector to a label that does not exist:

```yaml
selector:
  app: wrong
```

Apply:

```bash
kubectl apply -f nodeport.yaml
```

Check:

```bash
kubectl get endpoints nginx-nodeport
```

You should see no backend endpoints.

Now check:

```bash
kubectl get pods --show-labels
kubectl describe svc nginx-nodeport
```

Fix the selector:

```yaml
selector:
  app: nginx
```

Apply again:

```bash
kubectl apply -f nodeport.yaml
```

Verify:

```bash
kubectl get endpoints nginx-nodeport
```

This demonstrates the most common Service troubleshooting issue:

```text
Service Selector
       ↕
Pod Labels
```

---

