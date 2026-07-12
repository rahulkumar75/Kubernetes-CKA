# Kubernetes Storage: StorageClass & Dynamic Provisioning

> **Purpose:** Revision notes with concepts, comparisons, and YAML examples

---

## Table of Contents

1. [Types of Storage Provisioning](#types-of-storage-provisioning)
2. [Static Provisioning](#static-provisioning)
3. [Dynamic Provisioning](#dynamic-provisioning)
4. [Why Use Dynamic Provisioning?](#why-use-dynamic-provisioning)
5. [StorageClass Components](#storageclass-components)
6. [How Dynamic Provisioning Works](#how-dynamic-provisioning-works)
7. [volumeBindingMode](#volumebindingmode)
8. [Demo Walkthroughs](#demo-walkthroughs)
   - [Demo 1 – High-Performance StorageClass](#demo-1--high-performance-storageclass)
   - [Demo 2 – Cost-Effective StorageClass](#demo-2--cost-effective-storageclass)
   - [Setting a Default StorageClass](#setting-a-default-storageclass)
9. [Common Gotchas](#common-gotchas)
10. [Quick Reference Cheatsheet](#quick-reference-cheatsheet)

---

## Types of Storage Provisioning

| Feature | Static Provisioning | Dynamic Provisioning |
|---|---|---|
| **Who creates PV?** | Storage Admin (manually) | Kubernetes (automatically) |
| **PVC requires StorageClass?** | No | Yes |
| **PV created in advance?** | ✅ Yes | ❌ No |
| **Best for** | Predictable, fixed workloads | Variable, on-demand workloads |
| **Cost optimization** | Low — risk of over-provisioning | High — provision only what's needed |

---

## Static Provisioning

Also called **Pre-Provisioning**.

### Workflow

```
Storage Admin → Creates PV (e.g., 80Gi, RWO)
Developer     → Creates PVC (e.g., 10Gi, RWO)
Kubernetes    → Binds PVC to PV (if conditions match)
Pod           → Mounts the PVC
```

### Binding Conditions (Both must be satisfied)

1. **Capacity:** `PVC storage ≤ PV storage` (e.g., 10Gi ≤ 80Gi ✅)
2. **Access Mode:** Must be identical (e.g., both `ReadWriteOnce`)

> ⚠️ The PVC requests a *slice* of the PV — the PV is **pre-created** before any workload requests it.

---

## Dynamic Provisioning

### Workflow

```
Developer → Creates PVC with a StorageClass reference
Kubernetes → Detects no matching PV exists
           → Calls the StorageClass Provisioner
           → Creates a new PV matching the PVC's capacity & access mode
           → Binds PVC ↔ PV automatically
Pod        → Mounts the PVC
```

> 💡 Analogous to a **C++ `std::vector`** — storage grows dynamically as needed, rather than being pre-allocated.

---

## Why Use Dynamic Provisioning?

Different workloads have different storage requirements. Instead of pre-provisioning every disk type manually, StorageClasses let you define **profiles** and provision on demand.

| StorageClass Profile | Use Case | Cost |
|---|---|---|
| High-Performance SSD | Databases, low-latency apps | 💰💰💰 High |
| Balanced Persistent Disk | General-purpose workloads | 💰💰 Medium |
| HDD / Cost-Optimized | Logs, backups, archival data | 💰 Low |

**Benefits:**
- ✅ No manual PV creation
- ✅ Cost optimization — volumes only exist when needed
- ✅ Volumes are deleted when no longer required (based on Reclaim Policy)
- ✅ Flexible — different workloads use different storage tiers

---

## StorageClass Components

A `StorageClass` object has four key fields:

### 1. Provisioner
Defines **how** storage is created. Maps Kubernetes to the underlying cloud or storage provider.

| Provider | Provisioner String |
|---|---|
| Google Cloud (GCE PD) | `kubernetes.io/gce-pd` |
| AWS EBS | `kubernetes.io/aws-ebs` |
| Azure Disk | `kubernetes.io/azure-disk` |
| NFS | `kubernetes.io/nfs` (or external) |

### 2. Parameters
Configuration passed to the provisioner — controls performance, durability, and cost.

Examples:
- `type: pd-ssd` → SSD persistent disk
- `type: pd-standard` → HDD persistent disk
- `replication-type: regional-pd` → Cross-zone replication

### 3. Reclaim Policy
Defines what happens to the PV when the PVC is deleted.

| Policy | Behaviour |
|---|---|
| `Delete` (default) | PV and underlying storage are deleted |
| `Retain` | PV is kept; must be manually reclaimed |

### 4. volumeBindingMode
*(Covered in detail [below](#volumebindingmode))*

---

## How Dynamic Provisioning Works

```
PVC Created (with storageClassName)
        │
        ▼
StorageClass found?
   Yes ──► Provisioner invoked (e.g., GCE, AWS EBS)
        │
        ▼
   Volume Plugin maps K8s cluster ↔ Cloud Provider
        │
        ▼
   New PV created (matching PVC capacity & access mode)
        │
        ▼
   PVC ↔ PV Bound
        │
        ▼
   Pod mounts PVC
```

---

## volumeBindingMode

Controls **when** PV binding and dynamic provisioning occur.

| Mode | Behaviour |
|---|---|
| `Immediate` | PV is created and bound as soon as the PVC is created, even without a pod. |
| `WaitForFirstConsumer` | PV creation is deferred until a pod that uses the PVC is scheduled. Useful for **zone-aware** provisioning. |

> ⚠️ **`volumeBindingMode` is immutable.** You cannot edit it on an existing StorageClass.  
> To change it: delete the StorageClass → delete the associated PVC → recreate both.

---

## Demo Walkthroughs

### Demo 1 – High-Performance StorageClass

**StorageClass definition:**

```yaml
# high-performance-sc.yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: high-performance
provisioner: kubernetes.io/gce-pd
parameters:
  type: pd-ssd
volumeBindingMode: WaitForFirstConsumer
reclaimPolicy: Delete
```

**PVC using the StorageClass:**

```yaml
# prod-pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: prod-app-storage
  labels:
    environment: production
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: high-performance
  resources:
    requests:
      storage: 50Gi
```

> ⏳ With `WaitForFirstConsumer`, this PVC will remain in **Pending** state until a pod referencing it is scheduled.

---

### Demo 2 – Cost-Effective StorageClass

```yaml
# cost-effective-sc.yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: cost-effective
provisioner: kubernetes.io/gce-pd
parameters:
  type: pd-standard
  zones: us-central1-a      # Single zone = lower cost
volumeBindingMode: Immediate
reclaimPolicy: Delete
```

> Single-zone provisioning reduces cost because no cross-zone replication is used.

---

### Setting a Default StorageClass

Only **one** StorageClass should be marked as default at a time.

**Make `cost-effective` the default via patch:**

```bash
kubectl patch storageclass cost-effective \
  -p '{"metadata": {"annotations": {"storageclass.kubernetes.io/is-default-class": "true"}}}'
```

**Remove default from another StorageClass:**

```bash
kubectl patch storageclass standard-rwo \
  -p '{"metadata": {"annotations": {"storageclass.kubernetes.io/is-default-class": "false"}}}'
```

**Verify:**

```bash
kubectl get storageclass
# NAME                        PROVISIONER             AGE
# cost-effective (default)    kubernetes.io/gce-pd    5m
# high-performance            kubernetes.io/gce-pd    10m
```

> 💡 When a PVC is created **without specifying a `storageClassName`**, Kubernetes automatically uses the **default** StorageClass.

---

## Common Gotchas

| Issue | Cause | Fix |
|---|---|---|
| PVC stuck in `Pending` | `volumeBindingMode: WaitForFirstConsumer` — no pod has requested it yet | Create a pod that mounts the PVC |
| PVC stuck in `Pending` | No matching StorageClass found | Verify `storageClassName` in PVC matches an existing SC |
| Can't edit StorageClass | `volumeBindingMode` is **immutable** | Delete SC + PVC → recreate with correct mode |
| Two default StorageClasses | Multiple SCs annotated as default | Remove annotation from all but one |
| Old PV not cleaned up | PV not deleted after demo/test | Run `kubectl delete pv <pv-name>` manually |

---

## Quick Reference Cheatsheet

```bash
# List all StorageClasses
kubectl get storageclass

# Describe a StorageClass
kubectl describe storageclass <name>

# List PVCs and their status
kubectl get pvc

# List PVs
kubectl get pv

# Describe a PVC (useful for debugging Pending state)
kubectl describe pvc <name>

# Delete a StorageClass
kubectl delete storageclass <name>

# Patch StorageClass to set as default
kubectl patch storageclass <name> \
  -p '{"metadata":{"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```

---

## Summary

```
Static Provisioning
  Admin creates PV → Dev creates PVC → Bound → Pod uses it

Dynamic Provisioning
  Dev creates PVC (with StorageClass) → K8s creates PV automatically → Bound → Pod uses it

StorageClass = Blueprint for dynamic PV creation
  └── Provisioner    → which cloud/storage system
  └── Parameters     → performance/cost config
  └── ReclaimPolicy  → what happens when PVC is deleted
  └── BindingMode    → Immediate or WaitForFirstConsumer
```

---

*End of Notes: StorageClass & Dynamic Provisioning*