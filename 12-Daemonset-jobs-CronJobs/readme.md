# DaemonSet, Job & CronJob

## 🎯 Objective

* Understand **DaemonSets** and when to use them.
* Create and manage a DaemonSet using YAML.
* Understand **DaemonSet vs Deployment**.
* Learn the purpose of **Jobs and CronJobs**.
* Create, run, and verify Jobs and CronJobs.
* Understand which workload type to use for different real-world scenarios.

---

# 1. What is a DaemonSet?

A **DaemonSet** ensures that a Pod runs on **each eligible Node** in the cluster.

Example:

```text
3 Nodes
  ↓
DaemonSet
  ↓
Pod on Node 1
Pod on Node 2
Pod on Node 3
```

If a new eligible Node is added:

```text
New Node
   ↓
DaemonSet automatically creates a Pod
```

If a Node is removed, its DaemonSet Pod is also removed with the Node.

> **Memory:**
> **DaemonSet = One Pod per eligible Node**

### Common use cases

* Node-level monitoring agents
* Log collection agents
* Network components
* Storage/network plugins

Examples include:

* `kube-proxy`
* `fluent-bit`
* Node monitoring agents
* CNI components such as Calico/Flannel components



---

# 2. DaemonSet vs Deployment

| Deployment                        | DaemonSet                           |
| --------------------------------- | ----------------------------------- |
| Runs a desired number of replicas | Runs a Pod on each eligible Node    |
| Example: 3 replicas               | Example: 1 Pod per Node             |
| Used for application workloads    | Used mainly for node-level services |
| Scaling is replica-based          | Scaling follows eligible Nodes      |

### 🧠 Quick Memory

> **Deployment → How many Pods?**
> **DaemonSet → Which Nodes need a Pod?**

A DaemonSet normally runs one Pod per eligible Node, but scheduling rules such as **node selectors, affinity, and taints/tolerations** can limit which Nodes are eligible.

---

# 3. DaemonSet — Implementation

Create `daemonset.yaml`:

```yaml
apiVersion: apps/v1
kind: DaemonSet

metadata:
  name: nginx-ds

spec:
  selector:
    matchLabels:
      app: nginx

  template:
    metadata:
      labels:
        app: nginx

    spec:
      containers:
        - name: nginx
          image: nginx:latest
          ports:
            - containerPort: 80
```

Apply:

```bash
kubectl apply -f daemonset.yaml
```

Check:

```bash
kubectl get daemonset
kubectl get pods -o wide
```

You should see one DaemonSet Pod scheduled on each eligible Node.

Check details:

```bash
kubectl describe daemonset nginx-ds
```

---

# 4. Test DaemonSet Behavior

Check Nodes:

```bash
kubectl get nodes
```

Check Pods:

```bash
kubectl get pods -o wide
```

The Pods should be distributed across the eligible Nodes.

If a new eligible Node joins:

```text
New Node
   ↓
DaemonSet detects Node
   ↓
Creates DaemonSet Pod
```

---

# 5. Job

A **Job** creates Pods that run a task and **complete successfully**.

Unlike a Deployment, a Job is not intended to keep an application running continuously.

### Example use cases

* Database migration
* One-time data processing
* Backup
* Batch processing
* Initialization tasks

```text
Job
 ↓
Pod
 ↓
Task completes
 ↓
Pod = Completed
```

---

# 6. Job — Implementation

Create `job.yaml`:

```yaml
apiVersion: batch/v1
kind: Job

metadata:
  name: hello-job

spec:
  template:
    spec:
      restartPolicy: Never

      containers:
        - name: hello
          image: busybox:1.36
          command:
            - sh
            - -c
            - echo "Hello from Kubernetes Job"
```

Apply:

```bash
kubectl apply -f job.yaml
```

Check:

```bash
kubectl get jobs
kubectl get pods
```

Check output:

```bash
kubectl logs job/hello-job
```

Expected:

```text
Hello from Kubernetes Job
```

Once completed:

```text
Job → Completed
```

---

# 7. CronJob

A **CronJob creates Jobs according to a schedule**.

```text
CronJob
   ↓
Job
   ↓
Pod
   ↓
Task
```

Example use cases:

* Daily reports
* Scheduled backups
* Database cleanup
* Log cleanup
* Periodic batch processing

---

# 8. CronJob — Implementation

Create `cronjob.yaml`:

```yaml
apiVersion: batch/v1
kind: CronJob

metadata:
  name: hello-cronjob

spec:
  schedule: "*/1 * * * *"

  jobTemplate:
    spec:
      template:
        spec:
          restartPolicy: OnFailure

          containers:
            - name: hello
              image: busybox:1.36
              command:
                - sh
                - -c
                - date; echo "Hello from CronJob"
```

Apply:

```bash
kubectl apply -f cronjob.yaml
```

Check:

```bash
kubectl get cronjob
kubectl get jobs
kubectl get pods
```

After the scheduled time, you should see a new Job and Pod.

Check logs:

```bash
kubectl logs <pod-name>
```

---

# 9. Cron Syntax

The basic format is:

```text
* * * * *
│ │ │ │ │
│ │ │ │ └── Day of week
│ │ │ └──── Month
│ │ └────── Day of month
│ └──────── Hour
└────────── Minute
```

Examples:

```text
*/5 * * * *      → Every 5 minutes
0 * * * *        → Every hour
0 0 * * *        → Every day at midnight
0 0 * * 0        → Every Sunday at midnight
```

---

# 10. Job vs CronJob vs DaemonSet

| Workload      | Purpose                   | Example              |
| ------------- | ------------------------- | -------------------- |
| **DaemonSet** | Pod on each eligible Node | Log/monitoring agent |
| **Job**       | Run task to completion    | Database migration   |
| **CronJob**   | Run Jobs on a schedule    | Daily backup         |

### 🧠 Final Memory

```text
DaemonSet
   ↓
Every eligible Node

Job
   ↓
Run once → Complete

CronJob
   ↓
Schedule → Job → Complete
```

> **CKA Rule:**
> **Long-running application → Deployment**
> **One Pod per Node → DaemonSet**
> **One-time task → Job**
> **Scheduled task → CronJob**
