**FULL HANDS-ON CKA Question: Create user dev-user using CSR YAML + RBAC + kubeconfig** (very common exam task).

Perfect. Here is a **real CKA-style hands-on lab** for **CSR + RBAC + kubeconfig** (very common exam pattern).

---

# 🎯 Task

Create a user:

* Username: `dev-user`
* Group: `developers`

Give access to:

* namespace: `project-dev`
* permissions: get, list, create pods

Then configure kubeconfig for that user.

---

# ✅ Step 1: Generate Key + CSR

```bash id="kknp6x"
openssl genrsa -out dev-user.key 2048
```

```bash id="dtjwy8"
openssl req -new -key dev-user.key -out dev-user.csr -subj "/CN=dev-user/O=developers"
```

---

# ✅ Step 2: Encode CSR

```bash id="a42dc2"
cat dev-user.csr | base64 | tr -d '\n'
```

Copy output.

---

# ✅ Step 3: Create CSR YAML

```yaml id="44ekf7"
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: dev-user
spec:
  request: PASTE_BASE64_HERE
  signerName: kubernetes.io/kube-apiserver-client
  usages:
  - client auth
```

Save:

```bash id="yy6w91"
vi csr.yaml
```

Apply:

```bash id="l7wn4v"
kubectl apply -f csr.yaml
```

---

# ✅ Step 4: Approve CSR

```bash id="8u22cq"
kubectl certificate approve dev-user
```

Check:

```bash id="nl0s5g"
kubectl get csr
```

---

# ✅ Step 5: Extract Certificate

```bash id="93vtyz"
kubectl get csr dev-user -o jsonpath='{.status.certificate}' | base64 -d > dev-user.crt
```

---

# ✅ Step 6: Create Namespace

```bash id="qj7fdx"
kubectl create ns project-dev
```

---

# ✅ Step 7: Create Role

```yaml id="dfvt7n"
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: project-dev
  name: pod-role
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get","list","create"]
```

Apply:

```bash id="zd4jha"
kubectl apply -f role.yaml
```

---

# ✅ Step 8: Bind Role to User

```yaml id="v6dz7n"
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: pod-rolebinding
  namespace: project-dev
subjects:
- kind: User
  name: dev-user
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-role
  apiGroup: rbac.authorization.k8s.io
```

Apply:

```bash id="e6mffw"
kubectl apply -f rolebinding.yaml
```

---

# ✅ Step 9: Create kubeconfig for User

```bash id="i4yw25"
kubectl config set-credentials dev-user \
--client-key=dev-user.key \
--client-certificate=dev-user.crt \
--embed-certs=true
```

```bash id="s6o8nk"
kubectl config set-context dev-user-context \
--cluster=kubernetes \
--namespace=project-dev \
--user=dev-user
```

```bash id="pw4v7v"
kubectl config use-context dev-user-context
```

---

# ✅ Step 10: Verify

```bash id="pbj1lg"
kubectl auth can-i create pods
```

Expected:

```bash id="1icb2v"
yes
```

```bash id="2c3ptn"
kubectl auth can-i delete pods
```

Expected:

```bash id="nlr9k4"
no
```

---

# 🧠 Exam Fast Memory Trick

### User Creation Flow:

```text
key → csr → yaml → approve → crt → role → bind → kubeconfig
```

---

# 🔥 Most Common CKA Mistake

Inside RoleBinding:

```yaml id="5rz2s8"
subjects:
- kind: User
  name: dev-user
```

NOT ServiceAccount.

---

# 🚀 If you want, I can also give you **CKA Shortcut Method (2-minute CSR task in exam)** used by toppers.
