# Kubernetes DaemonSet

**Official Documentation**

https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/

---

# What is a DaemonSet?

A **DaemonSet** ensures that **every (or selected) node** in a Kubernetes cluster runs **exactly one copy** of a Pod.

Whenever a new worker node joins the cluster, Kubernetes automatically creates the DaemonSet Pod on that node.

Similarly, if a node is removed from the cluster, Kubernetes automatically removes the corresponding Pod.

Unlike a Deployment, where you specify the number of replicas, a DaemonSet automatically maintains **one Pod per node**.

---

# Why Do We Need DaemonSets?

DaemonSets are commonly used for software that must run on **every node**.

Examples include:

- Antivirus agents
- Malware scanning agents
- Log collection agents
- Monitoring agents
- Metrics collection agents
- Storage plugins
- Networking plugins

---

# Real-World Examples

## Antivirus Agent

Every worker node should run an antivirus process.

```
Node 1 → Antivirus Pod
Node 2 → Antivirus Pod
Node 3 → Antivirus Pod
```

---

## Malware Scanner

Each node continuously scans the local filesystem for malware.

```
Node 1 → Malware Scanner
Node 2 → Malware Scanner
Node 3 → Malware Scanner
```

---

## Log Collection Agent

Applications running on nodes generate logs.

A DaemonSet ensures every node has a log collector (such as Fluentd or Fluent Bit) to forward logs to a central logging system.

```
Node 1
 ├── Application Pods
 └── Log Collector

Node 2
 ├── Application Pods
 └── Log Collector

Node 3
 ├── Application Pods
 └── Log Collector
```

---

## Monitoring & Metrics Collection

Monitoring agents (such as Prometheus Node Exporter) collect:

- CPU usage
- Memory usage
- Disk usage
- Network statistics

Each node runs its own monitoring Pod.

---

# DaemonSet Architecture

```
                DaemonSet
                    │
      ┌─────────────┼─────────────┐
      │             │             │
      ▼             ▼             ▼
   Worker 1      Worker 2      Worker 3
      │             │             │
      ▼             ▼             ▼
  Agent Pod     Agent Pod     Agent Pod
```

One Pod is automatically created on every node.

---

# DaemonSet Manifest

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: anti-virus

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
        image: nginx
```

---

# Deploy the DaemonSet

```bash
kubectl apply -f daemonset-custom.yaml
```

---

# Verify the DaemonSet

```bash
kubectl get daemonset
```

Example output:

```text
NAME         DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
anti-virus   2         2         2       2            2           <none>          33s
```

### Understanding the Output

| Column | Meaning |
|---------|----------|
| DESIRED | Number of Pods that should exist (one per node) |
| CURRENT | Number of Pods currently running |
| READY | Number of Pods that are ready |
| UP-TO-DATE | Pods using the latest DaemonSet configuration |
| AVAILABLE | Pods available to serve workloads |

---

# Verify the Pods

```bash
kubectl get pods
```

Example output:

```text
NAME               READY   STATUS
anti-virus-2hbc9   1/1     Running
anti-virus-vdss6   1/1     Running
```

Notice there are **2 Pods**, one running on each node.

---

# Verify Cluster Nodes

```bash
kubectl get nodes
```

Example output:

```text
NAME           STATUS   ROLES
controlplane   Ready    control-plane
node01         Ready    <none>
```

Since the cluster has **2 nodes**, the DaemonSet creates **2 Pods**.

If another worker node joins later, Kubernetes automatically creates another DaemonSet Pod on that node.

---

# Deployment vs DaemonSet

| Deployment | DaemonSet |
|------------|-----------|
| User specifies number of replicas | Kubernetes creates one Pod per node |
| Used for applications | Used for node-level services |
| Pods can run on any node | One Pod runs on every node (or selected nodes) |
| Replica count is configurable | Replica count automatically matches eligible nodes |

---

# Useful Commands

List DaemonSets:

```bash
kubectl get daemonset
```

or

```bash
kubectl get ds
```

Describe a DaemonSet:

```bash
kubectl describe daemonset anti-virus
```

View Pods:

```bash
kubectl get pods -o wide
```

View Nodes:

```bash
kubectl get nodes
```

Delete the DaemonSet:

```bash
kubectl delete -f daemonset-custom.yaml
```

---

# Summary

- A **DaemonSet** ensures that every eligible node runs exactly one copy of a Pod.
- New nodes automatically receive a DaemonSet Pod.
- Removing a node also removes its DaemonSet Pod.
- Common use cases include:
  - Antivirus agents
  - Malware scanners
  - Log collection agents
  - Monitoring agents
  - Metrics collection agents
  - Storage and networking plugins
- Use `kubectl get daemonset` (or `kubectl get ds`) to view DaemonSets in the cluster.
