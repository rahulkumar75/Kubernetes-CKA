 # 📑 Node Affinity in Kubernetes

## 🎯 Objective

* Understand **Node Affinity** and how it controls Pod placement.
* Learn the difference between **required** and **preferred** affinity.
* Understand how Node Affinity differs from **Taints & Tolerations**.
* Practice scheduling Pods using **node labels + affinity**.
* Combine **Node Affinity + Taints/Tolerations** for controlled placement.

---

# 1. What is Node Affinity?

**Node Affinity** allows you to control which Nodes a Pod can be scheduled on based on **Node labels**.

It is a more flexible alternative to `nodeSelector`.



```text
Pod
 ↓
Node Affinity rules
 ↓
Match Node labels
 ↓
Eligible Node
```

### Common use cases

* GPU / AI workloads
* SSD or high-performance Nodes
* Environment-specific workloads
* Dedicated infrastructure
* Compliance-sensitive workloads

Example:

![without-node-affinity](/15-NodeAffinity/images/image-6.png)
- We overcome the limitations as shown in the picture.

![node-affinity](/15-NodeAffinity/images/image-7.png)

![node-affinity-2](/15-NodeAffinity/images/image-8.png)
- Finally, the pod is scheduled.

## In case someone removes the label value:
- If the label value is blank.

![no-labels-with-affinity](/15-NodeAffinity/images/image-9.png)

- So, in the case of Taints & Tolerance, it could be affected.
- But in the case of node affinity, we can manage this by using either of the two properties.

- There are two properties…
![alt text](/15-NodeAffinity/images/image-10.png)


1. **Required** during Scheduling, Ignore During Execution:
- In this case, the Pod label value needs to **match** with Node label (operator).

2. **Preferred** during Scheduling, Ignore During Execution:
- In this case, the Pod label value, even if not matched with the Node label value, will cause the pod will be scheduled. (Even then, it will be scheduled)

- **Last part says: ignored during execution** which means here, pod has already been scheduled on the node. Even after that, if there has been a change at the node level or anything, it won't impact the existing pod.

- Those existing pods will keep on running; it will only **impact the newer pods** that are yet to be schedule (or) that pod, which we will be scheduling after set the affinity.

- And the difference between those two is only **required** during scheduling and **preferred** during scheduling


---

# 2. Node Affinity vs Taints & Tolerations

| Node Affinity                  | Taints & Tolerations          |
| ------------------------------ | ----------------------------- |
| Attracts/selects Pods to Nodes | Repels unwanted Pods          |
| Rules are defined on the Pod   | Taint is defined on the Node  |
| Can enforce Node selection     | Toleration only allows access |
| Uses Node labels               | Uses taints/tolerations       |

### 🧠 Easy Memory

> **Taint → Node says NO**
> **Toleration → Pod says I CAN**
> **Affinity → Pod says I WANT**

A toleration alone does **not** tell Kubernetes which Node to choose.

---

# 3. Types of Node Affinity

## Required — Hard Rule

```yaml
requiredDuringSchedulingIgnoredDuringExecution:
```

The Pod can be scheduled **only on Nodes matching the rule**.

Example:

```yaml
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
        - matchExpressions:
            - key: disktype
              operator: In
              values:
                - ssd
```

Meaning:

```text
disktype=ssd → eligible
disktype=hdd → not eligible
```

If no matching Node is available, the Pod remains `Pending`.

---

## Preferred — Soft Rule

```yaml
preferredDuringSchedulingIgnoredDuringExecution:
```

The Scheduler **prefers** matching Nodes but can use another suitable Node if necessary.

```yaml
affinity:
  nodeAffinity:
    preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 100
        preference:
          matchExpressions:
            - key: disktype
              operator: In
              values:
                - ssd
```

Meaning:

> Prefer SSD Nodes, but fallback is allowed.

---

# 4. `IgnoredDuringExecution`

Both common affinity types use:

```text
IgnoredDuringExecution
```

It means the rule is evaluated during **scheduling**, but a later Node-label change does not cause the already-running Pod to be evicted because of that affinity rule.

Example:

```text
Pod scheduled
     ↓
Node has disktype=ssd
     ↓
Label removed
     ↓
Existing Pod → keeps running
New Pod      → affinity evaluated again
```

### 🧠 CKA Recall

> **Required = Must match when scheduling**
> **Preferred = Try to match when scheduling**
> **IgnoredDuringExecution = Don't evict because the label later changed**

---

# 5. Node Affinity Operators

| Operator       | Meaning                                             |
| -------------- | --------------------------------------------------- |
| `In`           | Label value must be one of the specified values     |
| `NotIn`        | Label value must not be one of the specified values |
| `Exists`       | Label key must exist                                |
| `DoesNotExist` | Label key must not exist                            |
| `Gt`           | Numeric value must be greater than                  |
| `Lt`           | Numeric value must be less than                     |

Example:

```yaml
operator: Exists
```

means only the **presence of the label key** matters.

---

# 🧪 6. Hands-on Implementation

## Step 1 — Check Nodes

```bash
kubectl get nodes
kubectl get nodes --show-labels
```

Choose a Node for the lab.

---

## Step 2 — Add a Label

```bash
kubectl label node <node-name> disktype=ssd
```

Verify:

```bash
kubectl get nodes --show-labels
```

---

## Step 3 — Create a Pod with Required Affinity

Create `affinity-pod.yaml`:

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: nginx-affinity

spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
              - key: disktype
                operator: In
                values:
                  - ssd

  containers:
    - name: nginx
      image: nginx
```

Apply:

```bash
kubectl apply -f affinity-pod.yaml
```

Verify placement:

```bash
kubectl get pod nginx-affinity -o wide
```

The Pod should run on a Node with:

```text
disktype=ssd
```

---


# 7. Test the Affinity Rule

Remove the label:

```bash
kubectl label node <node-name> disktype-
```

Check the existing Pod:

```bash
kubectl get pod nginx-affinity -o wide
```

The existing Pod continues running.

Now create another Pod using the same affinity:

```bash
kubectl apply -f affinity-pod.yaml
```

Check:

```bash
kubectl get pods -o wide
```

If no matching Node exists, the new Pod remains:

```text
Pending
```

### 🧪 Preferred Node Affinity Lab

Create `preferred-affinity-pod.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-preferred
spec:
  affinity:
    nodeAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
        - weight: 100
          preference:
            matchExpressions:
              - key: disktype
                operator: In
                values:
                  - ssd
  containers:
    - name: nginx
      image: nginx
```

Apply:

```bash
kubectl apply -f preferred-affinity-pod.yaml
```

Check placement:

```bash
kubectl get pod nginx-preferred -o wide
```

### Test fallback behavior

Remove the `disktype=ssd` label:

```bash
kubectl label node <node-name> disktype-
```

Create the Pod again:

```bash
kubectl delete pod nginx-preferred
kubectl apply -f preferred-affinity-pod.yaml
```

Check:

```bash
kubectl get pod nginx-preferred -o wide
```

The Pod can still be scheduled on another suitable Node because **Preferred Affinity is a soft rule**.

🧠 **CKA:** 
```
 `required` = must match → otherwise `Pending`
 `preferred` = try to match → fallback allowed
```


---

# 8. Combine Taints + Toleration + Affinity

For a dedicated GPU Node, you may want:

```text
                  GPU Node
                     │
          ┌──────────┴──────────┐
          │                     │
       Taint                 Label
    gpu=true               hardware=gpu
          │                     │
          ↓                     ↓
   Toleration              Affinity
          │                     │
          └──────────┬──────────┘
                     ↓
                 GPU Pod
```

### Why combine them?

**Taint:**

> Prevent unwanted Pods from using the Node.

**Toleration:**

> Allow the intended Pod to use the Node.

**Node Affinity:**

> Ensure the Pod selects the correct Node.

This gives stronger placement control than using either mechanism alone.

---

# 🔧 9. Troubleshooting

If a Pod is stuck in `Pending`:

```bash
kubectl get pod <pod-name>
kubectl describe pod <pod-name>
```

Check the **Events** section.

Then verify Node labels:

```bash
kubectl get nodes --show-labels
```

Check the affinity configuration:

```bash
kubectl get pod <pod-name> -o yaml
```

### Common causes

* Required label does not exist.
* Label key/value is incorrect.
* Node affinity rule is incorrect.
* Node has a taint without a matching toleration.
* Node does not have enough resources.

### CKA debugging pattern

```text
Pod Pending
    ↓
kubectl describe pod
    ↓
Check Events
    ↓
Check Node labels
    ↓
Check affinity
    ↓
Check taints/tolerations
```

---

# 🧠 Final CKA Recall

```text
nodeSelector
→ Simple Node selection

Node Affinity
→ Advanced Node selection

Required
→ Must match

Preferred
→ Best effort

Taint
→ Repels Pods

Toleration
→ Allows Pod onto tainted Node

Affinity
→ Selects desired Node
```

> **Most important memory:**
> **Taints/Tolerations answer "Can this Pod use the Node?"**
> **Node Affinity answers "Which Node should this Pod use?"**
