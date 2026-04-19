If you mean **Kubernetes CSR YAML file** (CertificateSigningRequest), here is a standard example used in **CKA practice**:

```yaml
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: rahul-csr
spec:
  request: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0K...
  signerName: kubernetes.io/kube-apiserver-client
  usages:
    - client auth
  expirationSeconds: 86400
```

## Important Part Breakdown

### 1. `request:`

This is the **base64 encoded CSR content** generated from OpenSSL.

Example command:

```bash
openssl req -new -key rahul.key -out rahul.csr -subj "/CN=rahul/O=dev"
```

Then encode:

```bash
cat rahul.csr | base64 | tr -d '\n'
```

Paste output inside `request:`.

---

### 2. `CN` and `O`

```bash
/CN=rahul/O=dev
```

Means:

* **CN** = username = `rahul`
* **O** = group = `dev`

Used for RBAC access.

---

### 3. Apply CSR

```bash
kubectl apply -f csr.yaml
```

Check:

```bash
kubectl get csr
```

Approve:

```bash
kubectl certificate approve rahul-csr
```

---

### 4. Get Certificate

```bash
kubectl get csr rahul-csr -o jsonpath='{.status.certificate}' | base64 -d
```

---

# CKA Exam Quick Template

```yaml
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: user1
spec:
  request: <base64 csr>
  signerName: kubernetes.io/kube-apiserver-client
  usages:
  - client auth
```

---
