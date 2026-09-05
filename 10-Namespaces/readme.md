# Kubernetes Namespaces

## 🎯 Objective

* Understand **what namespaces are and why they are used**.
* Learn to create, list, switch, and delete namespaces.
* Deploy and manage resources inside a specific namespace.
* Understand **namespace-scoped resource access**.
* Practice communication between Services in different namespaces using **DNS/FQDN**.
* Troubleshoot namespace-related issues using `kubectl`.

---

# 1. What is a Namespace?

A **Namespace** provides logical isolation for Kubernetes resources within a cluster.

For example:

```text
Kubernetes Cluster
│
├── default
│   ├── Deployment
│   ├── Service
│   └── Pods
│
└── demo
    ├── Deployment
    ├── Service
    └── Pods
```

Namespaces are useful for separating environments or teams:

```text
dev
staging
prod
```

> A Namespace is **not a separate Kubernetes cluster**. Resources in different namespaces still run on the same cluster.

---

# 2. Default Kubernetes Namespaces

Common namespaces include:

* `default` — default namespace for user workloads.
* `kube-system` — Kubernetes system components.
* `kube-public` — publicly readable cluster information.
* `kube-node-lease` — node heartbeat/lease information.

List namespaces:

```bash
kubectl get namespaces
```

or:

```bash
kubectl get ns
```

---

# 3. Working with Namespaces

### List resources in a namespace

```bash
kubectl get all -n kube-system
```

For another namespace:

```bash
kubectl get all -n <namespace>
```

> `kubectl get all` does not literally mean **every Kubernetes resource**. It shows a predefined set of common resources.

---

# 4. Specify Namespace Per Command

The simplest approach is:

```bash
kubectl get pods -n demo
kubectl get svc -n demo
kubectl get deployment -n demo
```

`-n` is short for:

```bash
--namespace
```

---

# 5. Set the Default Namespace

Instead of typing `-n demo` every time:

```bash
kubectl config set-context --current --namespace=demo
```

Now:

```bash
kubectl get pods
kubectl get svc
kubectl get deployment
```

will use the `demo` namespace by default.

### Check current namespace

```bash
kubectl config view --minify
```

Or:

```bash
kubectl config view --minify | grep namespace
```

### Switch back

```bash
kubectl config set-context --current --namespace=default
```

> **CKA Tip:** Always check your current context and namespace before running commands.

```bash
kubectl config current-context
kubectl config view --minify | grep namespace
```

---

# 6. Create a Namespace

## Imperative

```bash
kubectl create namespace demo
```

or:

```bash
kubectl create ns demo
```

Verify:

```bash
kubectl get ns
```

---

## Declarative

Create `namespace.yaml`:

```yaml
apiVersion: v1
kind: Namespace

metadata:
  name: demo
```

Apply:

```bash
kubectl apply -f namespace.yaml
```

### Common YAML Error

Incorrect:

```yaml
apiVersio: v1
```

Correct:

```yaml
apiVersion: v1
```

Example error:

```text
error validating "ns.yaml":
apiVersion not set, kind not set
```

**Fix the field name and apply again.**

---

# 7. Deploy an Application in a Namespace

Create a Deployment directly inside `demo`:

```bash
kubectl create deployment nginx-demo \
  --image=nginx \
  -n demo
```

Verify:

```bash
kubectl get deployment -n demo
kubectl get pods -n demo
```

Scale:

```bash
kubectl scale deployment nginx-demo \
  --replicas=3 \
  -n demo
```

Verify:

```bash
kubectl get pods -n demo
```

---

# 8. Expose the Deployment

Create a Service for the Deployment:

```bash
kubectl expose deployment nginx-demo \
  --name=svc-demo \
  --port=80 \
  --target-port=80 \
  -n demo
```

Verify:

```bash
kubectl get svc -n demo
kubectl get endpoints -n demo
```

---

# 9. Namespace-to-Namespace Communication

Pods in different namespaces can communicate through Services.

Suppose:

```text
demo namespace
    │
    └── svc-demo

default namespace
    │
    └── client Pod
```

A Service can be accessed using its DNS name.

### Same namespace

You can normally use:

```text
svc-demo
```

### Different namespace

Use:

```text
svc-demo.demo
```

### Full DNS name (FQDN)

```text
svc-demo.demo.svc.cluster.local
```

General format:

```text
<service>.<namespace>.svc.cluster.local
```

---

# 10. Hands-On Connectivity Test

Create a temporary client Pod:

```bash
kubectl run curl-client \
  --image=curlimages/curl \
  -it --rm \
  -- sh
```

From inside the Pod, test the Service:

```bash
curl http://svc-demo.demo.svc.cluster.local
```

You should receive the Nginx response.

This demonstrates:

```text
Pod
 │
 │ DNS
 ↓
svc-demo.demo.svc.cluster.local
 │
 ↓
Service
 │
 ↓
nginx-demo Pods
```

Exit:

```bash
exit
```

---


# 11. Important Namespace Concepts

### Namespace-scoped resources

Examples:

* Pods
* Deployments
* Services
* ConfigMaps
* Secrets

You must specify the namespace when accessing them if it is not the current namespace.

```bash
kubectl get pods -n demo
```

### Cluster-scoped resources

Examples:

* Nodes
* PersistentVolumes
* Namespaces
* ClusterRoles

These are not tied to a namespace.

For example:

```bash
kubectl get nodes
kubectl get pv
kubectl get namespaces
```

---


> **CKA Rule:** When something works in one namespace but not another, first verify the **namespace, Service, selector, Pod labels, and endpoints**.
