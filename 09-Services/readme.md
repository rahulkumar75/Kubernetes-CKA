I’ll restructure this as **Task 9 — Kubernetes Services**, keeping the implementation practical and CKA-focused. I’ll also correct the port terminology, Kind behavior, `ExternalName`, and YAML syntax issues. 

# Task 9 — Kubernetes Services

## 🎯 Objective

* Understand why Kubernetes **Services** are required for Pod-to-Pod and external communication.
* Learn the differences between **ClusterIP, NodePort, LoadBalancer, and ExternalName**.
* Understand **`port`, `targetPort`, and `nodePort`**.
* Create and verify Services using YAML and `kubectl`.
* Troubleshoot Services using **selectors, Endpoints, and Pod labels**.
* Understand how Service exposure works in a **Kind/local cluster**.

---

# 1. What is a Kubernetes Service?

Pods are **ephemeral** and their IP addresses can change when they are recreated.

A **Service** provides a stable endpoint to access a group of Pods.

```text
Client
   ↓
Service
   ↓
Pod 1
Pod 2
Pod 3
```

The Service uses a **selector** to identify the backend Pods.

> **Memory:**
> **Labels = identify Pods**
> **Selectors = find Pods**

---

# 2. Service Types

| Type             | Main Purpose                                    | Access           |
| ---------------- | ----------------------------------------------- | ---------------- |
| **ClusterIP**    | Internal application communication              | Inside cluster   |
| **NodePort**     | Expose application through node IP + port       | Outside cluster  |
| **LoadBalancer** | Expose application using external load balancer | External         |
| **ExternalName** | Access an external service using a DNS name     | External service |

> **Default Service type:** `ClusterIP`

---

# 3. Important Service Ports

This is important for CKA:

```text
Client
  ↓
nodePort: 30001        # NodePort Service
  ↓
port: 80               # Service port
  ↓
targetPort: 80         # Pod/container port
  ↓
Pod
```

### Remember

* **`port`** → Port exposed by the Service.
* **`targetPort`** → Port where the application is listening in the Pod.
* **`nodePort`** → Port exposed on each Node for a NodePort Service.

> `containerPort` in a Pod is mainly documentation/metadata; it does not itself expose the application.

---

# 4. NodePort

NodePort exposes a Service through:

```text
<NodeIP>:<NodePort>
```

Example:

```text
Node IP:     192.168.1.10
NodePort:    30001
Service:     80
Pod:         80
```

Traffic:

```text
External Client
      ↓
NodeIP:30001
      ↓
Service :80
      ↓
Pod :80
```

### NodePort range

The default Kubernetes NodePort range is:

```text
30000–32767
```

> Your original `30001–32767` range was slightly incorrect.

---

## NodePort YAML

```yaml
apiVersion: v1
kind: Service

metadata:
  name: nginx-nodeport

spec:
  type: NodePort

  selector:
    app: nginx

  ports:
    - port: 80
      targetPort: 80
      nodePort: 30001
```

Apply:

```bash
kubectl apply -f nodeport.yaml
```

Verify:

```bash
kubectl get svc
kubectl get nodes -o wide
kubectl describe svc nginx-nodeport
```

Check the backend Pods:

```bash
kubectl get pods --show-labels
kubectl get endpoints nginx-nodeport
```

---

# 5. NodePort with Kind

When using **Kind**, the Kubernetes nodes are Docker containers.

Therefore, accessing a NodePort from the host may require **port mapping** in the Kind cluster configuration.

Example:

```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4

nodes:
  - role: control-plane
    extraPortMappings:
      - containerPort: 30001
        hostPort: 30001
```

Then:

```text
Browser / curl
      ↓
localhost:30001
      ↓
Kind node
      ↓
NodePort Service
      ↓
Pod
```

Official Kind documentation:

[https://kind.sigs.k8s.io/docs/user/configuration/#extra-port-mappings](https://kind.sigs.k8s.io/docs/user/configuration/#extra-port-mappings)

> **Important:** Configure port mapping when creating the Kind cluster. If the existing cluster was created without the required mapping, recreating the cluster is often the simplest approach for a lab.

Test:

```bash
curl http://localhost:30001
```

---

# 6. ClusterIP

**ClusterIP** is the default Service type.

It provides a stable virtual IP and DNS name for **internal cluster communication**.

Example architecture:

```text
Frontend Pods
      ↓
backend-service:80
      ↓
Backend Pods
```

Instead of calling changing Pod IPs directly:

```text
10.244.1.5
10.244.2.7
10.244.1.9
```

the frontend uses:

```text
backend-service:80
```

Kubernetes automatically routes traffic to the Service's backend Pods.

---

## ClusterIP YAML

```yaml
apiVersion: v1
kind: Service

metadata:
  name: nginx-clusterip

spec:
  type: ClusterIP

  selector:
    app: nginx

  ports:
    - port: 80
      targetPort: 80
```

Apply:

```bash
kubectl apply -f clusterip.yaml
```

Verify:

```bash
kubectl get svc
kubectl describe svc nginx-clusterip
```

Check the selected Pods:

```bash
kubectl get endpoints nginx-clusterip
```

or:

```bash
kubectl get endpointslices
```

> **Modern Kubernetes:** EndpointSlices are the preferred API for representing Service endpoints, although `kubectl get endpoints` is still useful for CKA troubleshooting.

---

# 7. Service Selector → Endpoints

This is one of the most important concepts.

Suppose Pods have:

```yaml
labels:
  app: nginx
```

and the Service has:

```yaml
selector:
  app: nginx
```

Kubernetes finds matching Pods and adds their IPs as Service endpoints.

```text
Service
selector: app=nginx
       ↓
┌───────────────────┐
│ Pod 1             │
│ app=nginx         │
│ 10.244.1.5        │
└───────────────────┘
       ↓
┌───────────────────┐
│ Pod 2             │
│ app=nginx         │
│ 10.244.2.7        │
└───────────────────┘
```

If Pods are recreated and their IPs change, Kubernetes updates the endpoints automatically.

---

# 8. LoadBalancer

`LoadBalancer` is primarily used to expose a Service externally through a cloud/provider load balancer.

Typical flow:

```text
Internet
   ↓
Cloud Load Balancer
   ↓
NodePort / Service
   ↓
Pods
```

Example:

```yaml
apiVersion: v1
kind: Service

metadata:
  name: nginx-lb

spec:
  type: LoadBalancer

  selector:
    app: nginx

  ports:
    - port: 80
      targetPort: 80
```

Apply:

```bash
kubectl apply -f loadbalancer.yaml
```

Check:

```bash
kubectl get svc nginx-lb
```

### Why does `EXTERNAL-IP` remain `<pending>` locally?

A plain Kind cluster does not automatically provision a cloud load balancer.

Therefore, you may see:

```text
EXTERNAL-IP   <pending>
```

This does **not** mean the Service YAML is necessarily wrong.

In cloud environments such as EKS, GKE, or AKS, the cloud integration can provision an external load balancer.

> For local Kubernetes labs, `NodePort` is generally simpler for practicing external access.

---

# 9. ExternalName

`ExternalName` is different from the other Service types.

It does **not** select Pods and does not create a normal virtual IP.

Instead, it maps a Kubernetes Service name to an **external DNS name**.

Example:

```yaml
apiVersion: v1
kind: Service

metadata:
  name: external-db

spec:
  type: ExternalName
  externalName: database.example.com
```

A Pod can use:

```text
external-db
```

and DNS resolves it to:

```text
database.example.com
```

### Memory

```text
ClusterIP   → internal Pods
NodePort    → node IP + port
LoadBalancer → external load balancer
ExternalName → external DNS name
```

---

# 10. Service Troubleshooting

When a Service is not working, check these in order:

### 1. Check Service

```bash
kubectl get svc
```

### 2. Check selector

```bash
kubectl describe svc <service-name>
```

Look for:

```text
Selector: app=nginx
```

### 3. Check Pod labels

```bash
kubectl get pods --show-labels
```

The Service selector must match the Pod labels.

### 4. Check endpoints

```bash
kubectl get endpoints <service-name>
```

If there are **no endpoints**, common causes include:

* Selector does not match Pod labels.
* Pods are not Ready.
* Pods do not exist.

### 5. Check Pod status

```bash
kubectl get pods -o wide
kubectl describe pod <pod-name>
```

---

