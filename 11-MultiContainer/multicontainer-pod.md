# Multi-Container Pods in Kubernetes

Yes, a single Pod can run **multiple containers**.

Think of a Pod as a **shared environment**:

* Same Network Namespace (same IP)
* Same localhost
* Can share volumes
* Same lifecycle (scheduled together)

```text
Pod
│
├── Container A (nginx)
├── Container B (busybox)
│
├── Shared Network
├── Shared Storage
└── Shared Lifecycle
```

---

# 1. Why Multiple Containers?

### Single Container Pod

```text
Pod
└── Application
```

Example:

```text
Frontend Pod
└── Nginx
```

---

### Multi-Container Pod

```text
Pod
├── Application Container
└── Helper Container
```

Example:

```text
Pod
├── Nginx
└── Log Collector
```

---

# 2. Common Multi-Container Patterns

## Pattern 1: Sidecar

Most common.

```text
Pod
├── Main App
└── Sidecar
```

Example:

```text
Pod
├── Nginx
└── Fluent Bit
```

Flow:

```text
Application
     ↓
Writes Logs
     ↓
Shared Volume
     ↓
Fluent Bit
     ↓
ElasticSearch
```

Real-world:

* Log collection
* Monitoring agents
* Service mesh proxies (Istio Envoy)

---

## Pattern 2: Ambassador

Acts as a proxy.

```text
Pod
├── Application
└── Proxy Container
```

Example:

```text
Pod
├── App
└── HAProxy
```

```text
App
 ↓ localhost
HAProxy
 ↓
External DB
```

---

## Pattern 3: Adapter

Transforms data.

```text
Pod
├── Application
└── Adapter
```

Example:

```text
App Logs
     ↓
Adapter
     ↓
JSON Format
     ↓
Monitoring System
```

---

# 3. First Multi-Container Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multi-container
spec:
  containers:
  - name: nginx
    image: nginx

  - name: busybox
    image: busybox
    command:
    - sh
    - -c
    - "while true; do echo hello; sleep 5; done"
```

Create:

```bash
kubectl apply -f pod.yaml
```

Verify:

```bash
kubectl get pods
```

Output:

```text
multi-container   2/2 Running
```

Notice:

```text
2/2 Running
```

means:

```text
2 containers
2 healthy
```

---

# 4. See Containers Inside Pod

```bash
kubectl describe pod multi-container
```

Look under:

```text
Containers:
  nginx:
  busybox:
```

---

# 5. View Logs

By default Kubernetes asks which container.

### Nginx Logs

```bash
kubectl logs multi-container -c nginx
```

### Busybox Logs

```bash
kubectl logs multi-container -c busybox
```

Output:

```text
hello
hello
hello
```

---

# 6. Exec Into Specific Container

### Enter nginx

```bash
kubectl exec -it multi-container -c nginx -- sh
```

### Enter busybox

```bash
kubectl exec -it multi-container -c busybox -- sh
```

Verify hostname:

```bash
hostname
```

Both containers show same Pod hostname.

---

# 7. Shared Networking (Important Interview Topic)

Containers inside same pod share network.

```text
Pod IP = 10.244.0.5

nginx container
busybox container
```

Both use:

```text
10.244.0.5
```

---

## Verify

Inside busybox:

```bash
wget -qO- localhost
```

or

```bash
wget -qO- 127.0.0.1
```

Output:

```html
<html>
<head>...
```

Why?

```text
localhost
    ↓
nginx port 80
```

Containers communicate through localhost.

---

# 8. Shared Storage

Create shared volume.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: shared-volume
spec:
  volumes:
  - name: shared-data
    emptyDir: {}

  containers:
  - name: writer
    image: busybox
    command:
    - sh
    - -c
    - |
      while true
      do
        date > /data/time.txt
        sleep 5
      done
    volumeMounts:
    - name: shared-data
      mountPath: /data

  - name: reader
    image: busybox
    command:
    - sh
    - -c
    - |
      while true
      do
        cat /data/time.txt
        sleep 5
      done
    volumeMounts:
    - name: shared-data
      mountPath: /data
```

---

### Flow

```text
Writer Container
      ↓
Shared Volume
      ↓
Reader Container
```

---

Verify:

```bash
kubectl logs shared-volume -c reader
```

Output:

```text
Tue May 30 ...
Tue May 30 ...
```

---

# 9. How Kubernetes Tracks Multiple Containers

Check:

```bash
kubectl get pod multi-container
```

Output:

```text
READY   STATUS
2/2     Running
```

If one container crashes:

```text
READY   STATUS
1/2     Running
```

or

```text
1/2 CrashLoopBackOff
```

---

# 10. Container Restart Behavior

Suppose:

```text
Container A Running
Container B Crashed
```

Kubernetes restarts only:

```text
Container B
```

Not entire Pod.

Interview Question:

**Q: If one container crashes, does Kubernetes recreate the whole Pod?**

**Answer:** No. Kubernetes normally restarts only the failed container according to the Pod restart policy.

---

# 11. Real Production Example

```text
E-Commerce Pod
│
├── Node.js App
├── Fluent Bit
└── Istio Proxy
```

Responsibilities:

```text
Node.js
  ↓
Business Logic

Fluent Bit
  ↓
Log Shipping

Istio Proxy
  ↓
Traffic Management
```

---

# 12. Troubleshooting Commands

### List Containers

```bash
kubectl describe pod <pod-name>
```

### Container Logs

```bash
kubectl logs <pod-name> -c <container-name>
```

### Previous Logs

```bash
kubectl logs <pod-name> -c <container-name> --previous
```

### Exec

```bash
kubectl exec -it <pod-name> -c <container-name> -- sh
```

### Check Events

```bash
kubectl describe pod <pod-name>
```

---

# Frequently Asked Interview Questions (0–3 Years)

### Q1. Can a Pod have multiple containers?

Yes. A Pod can run one or more containers.

---

### Q2. Do containers inside a Pod have different IPs?

No.

All containers share the same Pod IP.

---

### Q3. How do containers communicate inside a Pod?

Using:

```text
localhost
127.0.0.1
```

---

### Q4. Can containers share files?

Yes.

Using shared volumes such as:

```text
emptyDir
PVC
hostPath
```

---

### Q5. What is a Sidecar Container?

A helper container that runs alongside the main application container.

Examples:

* Fluent Bit
* Envoy Proxy
* Monitoring Agents

---

### Q6. What are the advantages of Multi-Container Pods?

* Shared network
* Shared storage
* Tight coupling
* Easy localhost communication
* Sidecar architecture support

---

# Revision Diagram

```text
MULTI-CONTAINER POD

Pod
│
├── Container A (Main App)
├── Container B (Sidecar)
│
├── Shared IP
├── Shared localhost
├── Shared Volumes
└── Shared Lifecycle

Communication
│
├── localhost
├── 127.0.0.1
└── Shared Files

Patterns
│
├── Sidecar
├── Ambassador
└── Adapter

Commands
│
├── kubectl logs -c
├── kubectl exec -c
├── kubectl describe pod
└── kubectl get pod
```

For CKA and DevOps interviews, the most important concepts are:

1. Multi-container pod architecture
2. Sidecar pattern
3. Shared networking (`localhost`)
4. Shared volumes (`emptyDir`)
5. Troubleshooting with `logs` and `exec`
6. Init Containers vs Sidecar Containers (very commonly asked together)
