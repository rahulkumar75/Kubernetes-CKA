Since you're preparing for **CKA** and already have a **Kind cluster** plus an **AWS account**, I'd recommend learning this topic in **3 phases**:

1. **Phase 1:** Dynamic Provisioning on Kind (Local)
2. **Phase 2:** Volume Expansion
3. **Phase 3:** Real AWS EBS CSI Dynamic Provisioning (Production-like)

### Enterprise Scenario

A team deploys PostgreSQL via a StatefulSet. Developers only create PVCs. The cluster has a gp3 StorageClass using the AWS EBS CSI driver. Whenever a new replica is created, Kubernetes automatically provisions a new EBS volume, binds it to the PVC, and attaches it to the node running that Pod. This is exactly how dynamic provisioning works in production Kubernetes clusters.

---

# Lab 1: Dynamic Volume Provisioning on Kind

## Architecture

```text
Pod
 │
PVC
 │
StorageClass
 │
local-path-provisioner
 │
PV (auto-created)
 │
Host Disk
```

---

## Step 1: Verify Existing StorageClass

```bash
kubectl get sc
```

Expected:

```text
NAME                 PROVISIONER
standard (default)   rancher.io/local-path
```

If not present:

```bash
kubectl get pods -A | grep local-path
```

Kind often uses Local Path Provisioner.

---

## Step 2: Create Custom StorageClass

```yaml
# storageclass.yaml

apiVersion: storage.k8s.io/v1
kind: StorageClass

metadata:
  name: fast-storage

provisioner: rancher.io/local-path

reclaimPolicy: Delete

allowVolumeExpansion: true

volumeBindingMode: WaitForFirstConsumer
```

Apply:

```bash
kubectl apply -f storageclass.yaml
```

Verify:

```bash
kubectl get sc
```

---

## Step 3: Create PVC

```yaml
# pvc.yaml

apiVersion: v1
kind: PersistentVolumeClaim

metadata:
  name: mysql-pvc

spec:
  accessModes:
  - ReadWriteOnce

  storageClassName: fast-storage

  resources:
    requests:
      storage: 1Gi
```

Apply:

```bash
kubectl apply -f pvc.yaml
```

Check:

```bash
kubectl get pvc
```

Initially:

```text
Pending
```

Because:

```text
WaitForFirstConsumer
```

is enabled.

---

## Step 4: Deploy Pod

```yaml
# pod.yaml

apiVersion: v1
kind: Pod

metadata:
  name: storage-test

spec:
  containers:
  - name: app

    image: nginx

    volumeMounts:
    - name: data
      mountPath: /data

  volumes:
  - name: data
    persistentVolumeClaim:
      claimName: mysql-pvc
```

Apply:

```bash
kubectl apply -f pod.yaml
```

---

## Step 5: Observe Dynamic Provisioning

Check:

```bash
kubectl get pvc
kubectl get pv
```

Now:

```text
PVC -> Bound

PV -> Created Automatically
```

No manual PV creation.

---

## Step 6: Verify Data Persistence

Enter Pod:

```bash
kubectl exec -it storage-test -- sh
```

Create file:

```bash
echo "CKA Storage Lab" > /data/test.txt
```

Verify:

```bash
cat /data/test.txt
```

---

Delete Pod:

```bash
kubectl delete pod storage-test
```

Recreate:

```bash
kubectl apply -f pod.yaml
```

Check:

```bash
cat /data/test.txt
```

File still exists.

This proves:

```text
Container deleted
Storage survives
```

---

# Lab 2: Volume Expansion

---

## Current PVC

```bash
kubectl get pvc
```

Output:

```text
1Gi
```

---

## Expand

```bash
kubectl edit pvc mysql-pvc
```

Change:

```yaml
resources:
  requests:
    storage: 2Gi
```

Save.

---

Verify:

```bash
kubectl get pvc
```

Expected:

```text
2Gi
```

---

Check Events:

```bash
kubectl describe pvc mysql-pvc
```

Look for:

```text
FileSystemResizeSuccessful
```

---

# Lab 3: Reclaim Policy Testing

---

Check PV:

```bash
kubectl get pv
```

Describe:

```bash
kubectl describe pv
```

Notice:

```text
ReclaimPolicy: Delete
```

---

Delete PVC:

```bash
kubectl delete pvc mysql-pvc
```

Observe:

```bash
kubectl get pv
```

PV should disappear automatically.

---

## Change to Retain

Create new SC:

```yaml
reclaimPolicy: Retain
```

Create new PVC.

Delete PVC.

Observe:

```bash
kubectl get pv
```

PV remains.

This is common for:

```text
Production Databases
```

where data loss is unacceptable.

---

# Lab 4: Default StorageClass

Create:

```yaml
metadata:
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"
```

Apply.

Now create PVC without:

```yaml
storageClassName:
```

Kubernetes automatically chooses default StorageClass.

Verify:

```bash
kubectl describe pvc
```

---

# Lab 5 (Production-Like): AWS EBS CSI Driver

This is the most important real-world lab.

---

## Create EKS Cluster

Using:

```bash
eksctl create cluster \
--name storage-lab \
--region ap-south-1
```

---

## Install EBS CSI

```bash
eksctl create addon \
--name aws-ebs-csi-driver \
--cluster storage-lab
```

Verify:

```bash
kubectl get pods -n kube-system
```

Look for:

```text
ebs-csi-controller
```

---

## Create StorageClass

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass

metadata:
  name: ebs-gp3

provisioner: ebs.csi.aws.com

parameters:
  type: gp3

reclaimPolicy: Delete

volumeBindingMode: WaitForFirstConsumer

allowVolumeExpansion: true
```

Apply:

```bash
kubectl apply -f sc-ebs.yaml
```

---

## Create PVC

```yaml
apiVersion: v1
kind: PersistentVolumeClaim

metadata:
  name: ebs-pvc

spec:
  accessModes:
  - ReadWriteOnce

  storageClassName: ebs-gp3

  resources:
    requests:
      storage: 4Gi
```

Apply:

```bash
kubectl apply -f pvc-ebs.yaml
```

---

## Deploy Pod

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: ebs-test

spec:
  containers:
  - name: nginx
    image: nginx

    volumeMounts:
    - mountPath: /data
      name: storage

  volumes:
  - name: storage
    persistentVolumeClaim:
      claimName: ebs-pvc
```

---

Verify:

```bash
kubectl get pv
```

You will see:

```text
pvc-xxxxx
```

and in AWS:

```text
EC2
 └── Elastic Block Store
      └── New gp3 volume
```

automatically created.

---

# CKA Exam Checklist

Be able to do these from memory:

```bash
kubectl get sc
kubectl get pvc
kubectl get pv

kubectl describe pvc
kubectl describe pv

kubectl edit pvc

kubectl delete pvc
```

Understand:

```text
PV
PVC
StorageClass
Provisioner
CSI Driver
Dynamic Provisioning
Reclaim Policy
Volume Expansion
WaitForFirstConsumer
```


