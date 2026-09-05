## FQDN — Quick Recall

**FQDN = Fully Qualified Domain Name**

In Kubernetes, the FQDN gives the **complete DNS path to a Service**.

### Format

```text
<service-name>.<namespace>.svc.cluster.local
```

### Example

If:

```text
Service   = nginx-service
Namespace = demo
```

Then:

```text
nginx-service.demo.svc.cluster.local
```

### What each part means

```text
nginx-service   → Service name
demo            → Namespace
svc             → Kubernetes Service
cluster.local   → Cluster DNS domain
```

### When to use it

**Same namespace:**

```bash
curl http://nginx-service
```

**Different namespace:**

```bash
curl http://nginx-service.demo
```

**Full/explicit DNS name:**

```bash
curl http://nginx-service.demo.svc.cluster.local
```

### 🧠 FQDN Components

> **FQDN = Service + Namespace + svc + Cluster Domain**

```text
service.namespace.svc.cluster.local
```

**CKA shortcut:**
If a Pod needs to access a **Service in another namespace**, remember:

```text
<service>.<namespace>
```
---

## FQDN — Where & When Do You Use It?

FQDN is useful when you need to **reliably reach a Kubernetes Service using DNS**, especially when the Service is in **another namespace**.

### 1. Microservices in different namespaces ⭐

Real production example:

```text
frontend namespace
    │
    └── frontend-pod
            │
            │ needs API
            ↓
backend namespace
    │
    └── user-api Service
```

Frontend calls:

```bash
curl http://user-api.backend.svc.cluster.local
```

Why?

* `user-api` → Service name
* `backend` → Namespace
* Kubernetes DNS resolves it to the Service
* Service then routes traffic to the backend Pods

---

### 2. Same Service name in different namespaces

This is where FQDN becomes particularly useful.

You can have:

```text
dev/user-api
prod/user-api
```

From another namespace:

```text
user-api.dev.svc.cluster.local
user-api.prod.svc.cluster.local
```

Now there is **no ambiguity** about which Service you are accessing.

---

### 3. Production microservice communication

Imagine:

```text
frontend
   ↓
api-service
   ↓
payment-service
   ↓
database-service
```

If `payment-service` is in the `payments` namespace:

```text
payment-service.payments.svc.cluster.local
```

Your application can use this as its connection/endpoint.

Example environment variable:

```yaml
env:
  - name: PAYMENT_API
    value: "http://payment-service.payments.svc.cluster.local"
```

---

## When should YOU use FQDN?

| Scenario                          | Use                       |
| --------------------------------- | ------------------------- |
| Service in same namespace         | `service-name`            |
| Service in another namespace      | `service-name.namespace`  |
| Need completely explicit DNS name | **FQDN**                  |
| Microservices across namespaces   | **FQDN is a good choice** |
| Debugging DNS/connectivity        | **FQDN is very useful**   |

### 🧠 Future Recall

Think:

> **"I know the Service name, but I need to tell Kubernetes exactly WHERE that Service lives."**

Use:

```text
service.namespace.svc.cluster.local
```

**Real-world example:**

```text
frontend → user-api.backend.svc.cluster.local
```

That is the main scenario to remember for CKA + real-world DevOps.
