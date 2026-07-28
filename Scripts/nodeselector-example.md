# Kubernetes NodeSelector Example

## Objective

Learn how to use **nodeSelector** to schedule Pods onto specific Kubernetes nodes.

---

# What is nodeSelector?

A **nodeSelector** is the simplest scheduling constraint in Kubernetes.

It tells the scheduler:

> "Run this Pod only on a node that has a specific label."

One of the labels automatically assigned to every Kubernetes node is:

```text
kubernetes.io/hostname
```

This label contains the node's hostname.

---

# Lab Environment

Cluster Nodes

```bash
kubectl get nodes
```

Output

```text
NAME           STATUS   ROLES           AGE     VERSION
controlplane   Ready    control-plane   7d12h   v1.35.1
node01         Ready    <none>          7d12h   v1.35.1
```

---

# Pod Manifest

Create the following file.

**node.yaml**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: frontend
  labels:
    app: frontend
spec:
  containers:
  - name: frontend
    image: nginx
  nodeSelector:
    kubernetes.io/hostname: controlplane

---
apiVersion: v1
kind: Pod
metadata:
  name: frontendpod
  labels:
    app: frontend
spec:
  containers:
  - name: frontendpod
    image: nginx
  nodeSelector:
    kubernetes.io/hostname: node01
```

---

# Deploy the Pods

```bash
kubectl apply -f node.yaml
```

Output

```text
pod/frontend created
pod/frontendpod created
```

---

# Verify the Pods

```bash
kubectl get pods -o wide
```

Output

```text
NAME          READY   STATUS    RESTARTS   AGE   IP             NODE
frontend      1/1     Running   0          4s    192.168.0.55   controlplane
frontendpod   1/1     Running   0          4s    192.168.1.38   node01
```

Notice the **NODE** column.

| Pod | nodeSelector | Scheduled On |
|------|--------------|--------------|
| frontend | `controlplane` | controlplane |
| frontendpod | `node01` | node01 |

---

# How Kubernetes Scheduled These Pods

```
                    Kubernetes Cluster

        +---------------------------------------+

        +-------------------------------+
        | Control Plane Node            |
        | Hostname: controlplane        |
        |-------------------------------|
        | frontend                      |
        | nginx                         |
        +-------------------------------+

                    |

                    |

        +-------------------------------+
        | Worker Node                   |
        | Hostname: node01              |
        |-------------------------------|
        | frontendpod                   |
        | nginx                         |
        +-------------------------------+
```

The Kubernetes Scheduler checks the **nodeSelector** in each Pod and compares it with the labels available on each node.

If a matching label is found, the Pod is scheduled to that node.

---

# Verify the Node Labels

To see the labels on every node:

```bash
kubectl get nodes --show-labels
```

Or display only the hostname label:

```bash
kubectl get nodes --label-columns=kubernetes.io/hostname
```

Example

```text
NAME           STATUS   KUBERNETES.IO/HOSTNAME
controlplane   Ready    controlplane
node01         Ready    node01
```

The scheduler matches the value in `nodeSelector` with these labels.

---

# Scheduling Logic

For the first Pod:

```yaml
nodeSelector:
  kubernetes.io/hostname: controlplane
```

The scheduler searches for a node with:

```text
kubernetes.io/hostname=controlplane
```

Result:

```
✓ Scheduled on controlplane
```

For the second Pod:

```yaml
nodeSelector:
  kubernetes.io/hostname: node01
```

The scheduler searches for:

```text
kubernetes.io/hostname=node01
```

Result:

```
✓ Scheduled on node01
```

---

# What Happens if No Matching Node Exists?

Example

```yaml
nodeSelector:
  kubernetes.io/hostname: worker02
```

Since no node has this hostname label, Kubernetes cannot schedule the Pod.

The Pod remains in the **Pending** state.

```text
NAME      READY   STATUS
frontend  0/1     Pending
```

---

# Key Points

- `nodeSelector` is the simplest way to control where a Pod runs.
- It works by matching **node labels**.
- `kubernetes.io/hostname` is an automatically assigned label on every node.
- If no node matches the selector, the Pod remains **Pending**.
- The scheduler only places the Pod on nodes whose labels exactly match the `nodeSelector`.

---

# Useful Commands

View Nodes

```bash
kubectl get nodes
```

View Node Labels

```bash
kubectl get nodes --show-labels
```

Show Hostname Labels

```bash
kubectl get nodes --label-columns=kubernetes.io/hostname
```

Deploy Pods

```bash
kubectl apply -f node.yaml
```

View Pods

```bash
kubectl get pods
```

View Pods with Assigned Nodes

```bash
kubectl get pods -o wide
```

Describe a Pod

```bash
kubectl describe pod frontend
```

Delete the Pods

```bash
kubectl delete -f node.yaml
```

---

# Summary

A **nodeSelector** allows you to explicitly control which node a Pod is scheduled on by matching node labels. In this example:

- The **frontend** Pod runs on the **controlplane** node.
- The **frontendpod** Pod runs on **node01**.

This is the simplest and most commonly used method for basic Pod scheduling in Kubernetes.
