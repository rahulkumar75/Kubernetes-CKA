# HPA | Horizontal Pod Autoscaler

## 🎯 Objective

* Understand **Kubernetes autoscaling**.
* Learn the difference between **HPA, VPA, and Cluster Autoscaler**.
* Understand how HPA uses metrics to adjust Pod replicas.
* Configure HPA using CPU utilization.
* Generate load and observe automatic scale-out and scale-in.

---

# 1. What is Scaling?

**Scaling** means adjusting resources to handle changes in workload demand.

In Kubernetes, scaling can happen in different ways:

```text
Traffic / Workload increases
          ↓
      More capacity
          ↓
   ┌──────┴──────┐
   ↓             ↓
More Pods     More CPU/Memory
(Horizontal)  (Vertical)
```

### Example

A Deployment initially has:

```text
replicas: 2
```

If traffic increases, we may need:

```text
2 Pods → 4 Pods → 6 Pods
```

This is **horizontal scaling**.

---

# 2. Types of Autoscaling

## Horizontal Pod Autoscaling — HPA

HPA automatically changes the **number of Pod replicas** based on metrics.

![hpa](image.png)

```text
CPU / Memory / Custom Metric
          ↓
         HPA
          ↓
   Replica count changes
```

Example:

```text
2 Pods
  ↓
High CPU
  ↓
HPA
  ↓
4 Pods
```

### 🧠 Remember

> **HPA = More or fewer Pods**

---

## Vertical Pod Autoscaling — VPA

VPA adjusts the **CPU and memory requests/limits** of Pods based on observed resource usage.

```text
Current Pod
   ↓
Resource usage observed
   ↓
VPA recommendation/update
   ↓
CPU / Memory resources adjusted
```

VPA can require Pod restarts depending on how the resource recommendation is applied.

### 🧠 Remember

> **VPA = Resize Pod resources**

---

## Cluster Autoscaler

HPA creates more Pods, but those Pods still need Nodes to run on.

If the cluster does not have enough Node capacity:

```text
HPA
 ↓
More Pods
 ↓
Pods Pending
 ↓
Cluster Autoscaler
 ↓
Adds Nodes
```

Similarly, Cluster Autoscaler can remove underutilized Nodes when appropriate.

## Summary Scaling Type

![type-of-scalling](image-1.png)

- Horizontal auto scaling, also known as - Scale Out/IN
    - When we are scaling out, it means we are adding more replicas.
- Vertical Auto Scaling, also known as - Resizing the Machine, we **scale up/down** to meet the increased or decreased load
- HPA & VPA are done based on **Metrics**.

# Note: Who is responsible for autoscaling

> HPA: **It is Provided by the K8S.**
> 

> VPA: Needs to be installed externally.
> 

> Cluster autoscaler & node auto-provisioning are provided by the Cloud Provider.
> 

## Event-Based Auto Scaling (KEDA)

- It is done by a third-party tool, **KEDA.**

### 🧠 Remember

> **HPA scales Pods; Cluster Autoscaler scales Nodes.**

---

# 3. Scaling Types

| Type       | What changes?            | Example      |
| ---------- | ------------------------ | ------------ |
| Horizontal | Number of Pods           | 2 → 5 Pods   |
| Vertical   | Pod CPU/Memory resources | 500m → 1 CPU |
| Cluster    | Number of Nodes          | 3 → 5 Nodes  |

---

# 4. Metrics-Based Autoscaling

HPA can make scaling decisions using metrics such as:

* CPU utilization
* Memory utilization
* Custom metrics
* External metrics

For this lab, we will use **CPU utilization**.

> HPA requires a metrics source. In a typical basic lab, **Metrics Server** provides CPU/memory resource metrics.

---

# 🧪 5. Hands-on: HPA on Kind

## Step 1 — Verify the Cluster

```bash
kubectl get nodes
```

Check Metrics Server:

```bash
kubectl top nodes
```

If this command works, resource metrics are available.

Also verify:

```bash
kubectl get pods -n kube-system
```

Look for:

```text
metrics-server
```

---

# 6. Create a Namespace

```bash
kubectl create namespace hpa-demo
```

---

# 7. Create a CPU-Based Deployment

Create `hpa-demo.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: hpa-demo
  namespace: hpa-demo

spec:
  replicas: 1

  selector:
    matchLabels:
      app: hpa-demo

  template:
    metadata:
      labels:
        app: hpa-demo

    spec:
      containers:
        - name: app
          image: nginx

          resources:
            requests:
              cpu: "100m"
            limits:
              cpu: "500m"

          ports:
            - containerPort: 80
```

Apply:

```bash
kubectl apply -f hpa-demo.yaml
```

Verify:

```bash
kubectl get pods -n hpa-demo
```

---

# 8. Create a Service

```bash
kubectl expose deployment hpa-demo \
  --name=hpa-demo \
  --port=80 \
  --target-port=80 \
  -n hpa-demo
```

Verify:

```bash
kubectl get svc -n hpa-demo
```

---

# 9. Create the HPA

Create an HPA targeting **50% average CPU utilization**:

```bash
kubectl autoscale deployment hpa-demo \
  --cpu=50% \
  --min=1 \
  --max=5 \
  -n hpa-demo
```

Verify:

```bash
kubectl get hpa -n hpa-demo
```

You should see something similar to:

```text
NAME       TARGETS    MINPODS   MAXPODS   REPLICAS
hpa-demo   0%/50%     1         5         1
```

### What does `0%/50%` mean?

```text
Current CPU utilization / Target CPU utilization
```

Example:

```text
45% / 50%
```

means:

```text
Current average CPU = 45%
Target CPU           = 50%
```

It does **not** mean the Pod is using 45% of the Node's CPU.

HPA CPU utilization is calculated relative to the **CPU requests** of the containers.

---

# 🔥 10. Generate CPU Load

Open another terminal.

Create a temporary Pod that continuously sends requests to the application:

```bash
kubectl run load-generator \
  --image=busybox:1.36 \
  --restart=Never \
  -n hpa-demo \
  -- /bin/sh -c \
  "while true; do wget -q -O- http://hpa-demo; done"
```

Check the load:

```bash
kubectl top pods -n hpa-demo
```

You should see CPU usage increase.

---

# 11. Watch HPA Scaling

In another terminal:

```bash
kubectl get hpa -n hpa-demo --watch
```

Also watch Pods:

```bash
kubectl get pods -n hpa-demo --watch
```

You should observe the general flow:

```text
CPU usage increases
       ↓
HPA observes metric
       ↓
Current utilization > target
       ↓
Desired replicas increase
       ↓
Deployment creates more Pods
```

For example:

```text
1 Pod
 ↓
High CPU
 ↓
2 Pods
 ↓
High CPU continues
 ↓
3 Pods
```

The exact replica count and timing depend on the observed metrics and HPA control loop.

---

# 12. Stop the Load

Delete the load generator:

```bash
kubectl delete pod load-generator -n hpa-demo
```

Watch:

```bash
kubectl get hpa -n hpa-demo --watch
```

And:

```bash
kubectl get pods -n hpa-demo --watch
```

After CPU utilization falls and the HPA's scale-down behavior takes effect:

```text
High CPU
   ↓
More replicas

Load stops
   ↓
CPU decreases
   ↓
HPA scales down
   ↓
Fewer replicas
```

> Scale-down is not necessarily immediate. HPA uses stabilization and timing behavior to avoid rapidly adding/removing Pods.

---

# 13. Understand the Complete Flow

```text
                 User Traffic
                     ↓
              Application Pods
                     ↓
                CPU increases
                     ↓
              Metrics Server
                     ↓
                   HPA
                     ↓
          Desired replicas calculated
                     ↓
                Deployment
                     ↓
          More / fewer Pods created
```

If there is insufficient Node capacity:

```text
HPA
 ↓
More Pods required
 ↓
Pods Pending
 ↓
Cluster lacks capacity
 ↓
Cluster Autoscaler
 ↓
More Nodes
```

---

# 🔧 14. Troubleshooting

## HPA shows `<unknown>`

Check:

```bash
kubectl get hpa -n hpa-demo
```

Then:

```bash
kubectl describe hpa hpa-demo -n hpa-demo
```

Verify Metrics Server:

```bash
kubectl top pods -n hpa-demo
kubectl top nodes
```

Check:

```bash
kubectl get pods -n kube-system
```

---

## HPA is not scaling

Check:

```bash
kubectl describe hpa hpa-demo -n hpa-demo
```

Then verify:

```bash
kubectl top pods -n hpa-demo
```

Also check that the target Deployment's containers have a **CPU request** configured.

For CPU utilization-based HPA, the percentage is calculated relative to CPU requests.

---

## Pods are Pending after HPA scales

Check:

```bash
kubectl get pods -n hpa-demo
kubectl describe pod <pod-name> -n hpa-demo
```

Look at Events for:

```text
Insufficient cpu
Insufficient memory
```

This means:

```text
HPA successfully requested more Pods
             ↓
Cluster does not have enough capacity
             ↓
Pods remain Pending
```

This is where **Cluster Autoscaler** can become relevant in a cloud-managed cluster.

---

# 🧹 15. Cleanup

```bash
kubectl delete namespace hpa-demo
```

---

# 🧠 Final CKA Cheat Sheet

```text
HPA
→ Changes Pod replicas

VPA
→ Changes Pod resource requests/limits

Cluster Autoscaler
→ Changes Node count

HPA CPU target
→ Current utilization compared with target

45% / 50%
→ Current CPU utilization = 45%
→ Target = 50%

High CPU
→ HPA may scale out

Low CPU
→ HPA may scale in

HPA creates Pods
→ Scheduler places them on Nodes

No Node capacity
→ Pods may remain Pending
```

### One-line memory rule

> **HPA = Scale Pods | VPA = Resize Pods | Cluster Autoscaler = Scale Nodes**
