# Kubernetes Node Affinity

**Official Documentation**

- https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/

---

# What is Node Affinity?

**Node Affinity** is an advanced version of **Node Selector**.

Like `nodeSelector`, it allows you to control which worker nodes your Pods can run on, but it is **more flexible and expressive**.

For example:

- Run a Pod only on SSD nodes.
- Run a Pod on any node except Micro nodes.
- Prefer High-Memory nodes but allow other nodes if unavailable.
- Schedule Pods only on GPU-enabled nodes.

---

# Node Selector vs Node Affinity

| Node Selector | Node Affinity |
|---------------|---------------|
| Simple key=value matching | Supports multiple operators |
| Exact match only | Flexible matching rules |
| Hard requirement only | Supports hard and soft requirements |
| Less expressive | More powerful and recommended |

---

# Why Use Node Affinity?

Worker nodes may have different hardware configurations.

Example:

| Node | CPU | Memory | Disk |
|------|-----|--------|------|
| node01 | 2 CPU | 4 GB | HDD |
| node02 | 8 CPU | 16 GB | SSD |
| node03 | 32 CPU | 128 GB | SSD |

Some applications require:

- More CPU
- More Memory
- SSD Storage
- GPU Nodes
- Compliance or Security isolation

Node Affinity allows Kubernetes to schedule Pods accordingly.

---

# Operators Supported

| Operator | Description |
|----------|-------------|
| In | Matches nodes with the specified label values |
| NotIn | Excludes nodes with the specified label values |
| Exists | Matches nodes that contain the label key, regardless of value |
| DoesNotExist | Matches nodes that do not contain the label key |
| Gt | Numeric value greater than specified |
| Lt | Numeric value less than specified |

---

# Example Operators

### In

```yaml
operator: In
values:
- ssd
```

Meaning:

> Run only on nodes labeled `ssd`.

---

### NotIn

```yaml
operator: NotIn
values:
- micro
```

Meaning:

> Run on any node except Micro nodes.

---

### Exists

```yaml
operator: Exists
```

Meaning:

> Run on any node that has this label.

---

### DoesNotExist

```yaml
operator: DoesNotExist
```

Meaning:

> Do not schedule on nodes that contain this label.

---

### Gt / Lt

```yaml
operator: Gt
values:
- "8"
```

Meaning:

> Schedule only on nodes whose label value is greater than 8.

---

# Hard vs Soft Scheduling

Node Affinity supports two scheduling modes.

## 1. Required (Hard Requirement)

```text
requiredDuringSchedulingIgnoredDuringExecution
```

The Pod **must** be scheduled on a matching node.

If no node satisfies the condition:

- Pod remains Pending.

---

## 2. Preferred (Soft Requirement)

```text
preferredDuringSchedulingIgnoredDuringExecution
```

The scheduler tries to place the Pod on a matching node.

If no matching node exists:

- The Pod is still scheduled on another available node.

---

# Hard vs Soft Comparison

| Mode | Description |
|------|-------------|
| requiredDuringSchedulingIgnoredDuringExecution | Strict requirement. Pod will not be scheduled unless a matching node exists. |
| preferredDuringSchedulingIgnoredDuringExecution | Preferred placement. Pod can still run on another node if no match is found. |

---

# Example 1 - Required Node Affinity (Hard)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx

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

### What happens?

Only worker nodes with:

```text
disktype=ssd
```

can run this Pod.

If none exist:

```text
STATUS = Pending
```

---

# Example 2 - Preferred Node Affinity (Soft)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx

spec:
  affinity:
    nodeAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 1
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

Deploy:

```bash
kubectl apply -f nodeAffinity-preferred.yaml
```

### What happens?

Kubernetes:

1. Looks for SSD nodes.
2. If one exists, schedules the Pod there.
3. If none exist, schedules the Pod on another node.

The Pod **never remains Pending** because the rule is only a preference.

---

# Example 3 - Required Node Affinity

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: kplabs-node-affinity

spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: disk
            operator: In
            values:
            - ssd2

  containers:
  - name: with-node-affinity
    image: nginx
```

Since the Pod requires:

```text
disk=ssd2
```

it remains **Pending** until a node with that label exists.

---

# Example 4 - Preferred Node Affinity

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: kplabs-node-affinity-preferred

spec:
  affinity:
    nodeAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 1
        preference:
          matchExpressions:
          - key: memory
            operator: In
            values:
            - high
            - medium

  containers:
  - name: kplabs-affinity-preferred
    image: nginx
```

Kubernetes prefers nodes with:

```text
memory=high
```

or

```text
memory=medium
```

If neither label exists, the Pod is still scheduled on another available node.

---

# Adding Labels to Nodes

Label a worker node:

```bash
kubectl label node node01 disktype=ssd
```

Output:

```text
node/node01 labeled
```

Verify:

```bash
kubectl describe node node01
```

or

```bash
kubectl get nodes --show-labels
```

---

# Verify Pod Status

```bash
kubectl get pods
```

Example:

```text
NAME    READY   STATUS
nginx   1/1     Running
```

---

# Node Affinity Workflow

```
Create Pod
      │
      ▼
Read Node Affinity Rules
      │
      ▼
Search Worker Nodes
      │
      ├──────────────┐
      │              │
 Matching Node    No Match
      │              │
      ▼              ▼
Schedule Pod     Pending (Required)

             OR

Schedule on another node (Preferred)
```

---

# Pod Affinity

Pod Affinity schedules a Pod **near another Pod**.

Example:

```yaml
affinity:
  podAffinity:
```

Use Pod Affinity when two applications communicate frequently and should run on the same worker node.

Example:

```
Worker Node

Frontend
Backend
Redis
```

Keeping related Pods together reduces network latency.

---

# Pod Anti-Affinity

Pod Anti-Affinity does the opposite.

It tells Kubernetes:

> Do **not** place Pods together on the same node.

Example:

```
Node1
------
Pod A

Node2
------
Pod B

Node3
------
Pod C
```

This improves:

- High Availability
- Fault Tolerance
- Load Distribution

If one node fails, not all replicas are lost.

---

# Example Pod Affinity

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-kubernetes-second

spec:
  replicas: 3

  selector:
    matchLabels:
      app: hello-kubernetes-second

  template:
    metadata:
      labels:
        app: hello-kubernetes-second

    spec:
      affinity:
        podAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: NotIn
                values:
                - frontend
            topologyKey: kubernetes.io/hostname

      containers:
      - name: with-node-affinity
        image: nginx
```

> **Note:** Despite the heading "Node Anti Affinity", this manifest actually demonstrates **Pod Affinity**, because it uses `podAffinity`. If you wanted to spread Pods across different nodes, you would use `podAntiAffinity` instead.

---

# Useful Commands

Label a node:

```bash
kubectl label node node01 disktype=ssd
```

Remove a label:

```bash
kubectl label node node01 disktype-
```

View node labels:

```bash
kubectl get nodes --show-labels
```

Describe a node:

```bash
kubectl describe node node01
```

Deploy a Pod:

```bash
kubectl apply -f nodeAffinity-required.yaml
```

View Pods:

```bash
kubectl get pods -o wide
```

---

# Summary

- **Node Affinity** is a more powerful version of `nodeSelector`.
- It supports flexible scheduling using operators such as `In`, `NotIn`, `Exists`, `DoesNotExist`, `Gt`, and `Lt`.
- **Required** affinity is a hard rule. If no node matches, the Pod remains **Pending**.
- **Preferred** affinity is a soft rule. Kubernetes prefers matching nodes but can schedule the Pod elsewhere.
- **Pod Affinity** places related Pods on the same node.
- **Pod Anti-Affinity** spreads Pods across different nodes to improve availability and fault tolerance.
