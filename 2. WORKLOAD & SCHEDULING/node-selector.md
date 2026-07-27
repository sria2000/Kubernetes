# Kubernetes Node Selector

`nodeSelector` is the simplest scheduling mechanism in Kubernetes. It tells Kubernetes to run a Pod only on nodes that have a specific label.

---

# Why Do We Need Node Selectors?

In a Kubernetes cluster:

- Worker nodes can have **different CPU, Memory, and storage capacities**.
- Some applications require **more CPU and RAM** than others.
- Applications may need to be **segregated** for compliance, security, licensing, or performance.
- Node labels help Kubernetes identify suitable nodes for different workloads.

Example:

| Node | Capacity | Intended Workload |
|------|----------|-------------------|
| node01 | 2 CPU, 4 GB RAM | Development |
| node02 | 16 CPU, 64 GB RAM | Production |
| node03 | GPU | Machine Learning |

Instead of allowing Kubernetes to place Pods anywhere, you can tell it exactly which nodes are eligible.

---

# How Node Selector Works

The process is simple:

1. **Assign labels** to worker nodes.
2. **Use `nodeSelector`** inside the Pod specification.
3. Kubernetes schedules the Pod only on nodes whose labels match.

```
Worker Node
      │
      │  Label
      ▼
size=Large
      ▲
      │
nodeSelector
      │
      ▼
      Pod
```

---

# Step 1: Base Pod Manifest

Without a node selector, Kubernetes can schedule the Pod on any available worker node.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simplepod
spec:
  containers:
  - name: nginx-pod
    image: nginx
```

Deploy:

```bash
kubectl apply -f nodeSelector-pod.yaml
```

---

# Step 2: Add a Node Selector

Update the Pod manifest to specify the required node label.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: largepod
spec:
  nodeSelector:
    size: Large
  containers:
  - name: nginx-pod
    image: nginx
```

Apply the manifest:

```bash
kubectl apply -f nodeSelector-pod.yaml
```

---

# Step 3: Check the Pod Status

Since no node currently has the label `size=Large`, Kubernetes cannot schedule the Pod.

```bash
kubectl get pods -o wide
```

Output:

```text
NAME       READY   STATUS    RESTARTS   AGE   IP       NODE
largepod   0/1     Pending   0          70s   <none>   <none>
```

### Why is the Pod Pending?

The Pod requests:

```yaml
nodeSelector:
  size: Large
```

But no worker node has that label.

Therefore, Kubernetes keeps the Pod in the **Pending** state until a matching node becomes available.

---

# Step 4: Verify Existing Node Labels

List all nodes:

```bash
kubectl get nodes
```

Describe a node:

```bash
kubectl describe node node01
```

Since the label doesn't exist yet, you won't see:

```text
size=Large
```

---

# Step 5: Label the Worker Node

Assign the required label to the worker node.

```bash
kubectl label node node01 size=Large
```

Output:

```text
node/node01 labeled
```

---

# Step 6: Verify the Pod

After the node is labeled, Kubernetes immediately schedules the Pod.

```bash
kubectl get pods -o wide
```

Output:

```text
NAME       READY   STATUS    RESTARTS   AGE     IP             NODE
largepod   1/1     Running   0          3m15s   192.168.1.67   node01
```

Notice that the Pod is now running on **node01**.

---

# Step 7: Verify the Node Label

```bash
kubectl describe node node01 | grep -i size
```

Output:

```text
size=Large
```

This confirms that the node has the expected label.

---

# Removing the Label

To remove a label from a node:

```bash
kubectl label node node01 size-
```

Output:

```text
node/node01 unlabeled
```

---

# Delete the Pod

```bash
kubectl delete -f nodeSelector-pod.yaml
```

---

# Useful Commands

List nodes:

```bash
kubectl get nodes
```

Describe a node:

```bash
kubectl describe node node01
```

Add a label:

```bash
kubectl label node node01 size=Large
```

Remove a label:

```bash
kubectl label node node01 size-
```

View Pods with node information:

```bash
kubectl get pods -o wide
```

---

# Summary

- `nodeSelector` is the simplest way to schedule Pods onto specific worker nodes.
- Worker nodes are identified using **labels**.
- Pods specify the required label using the **nodeSelector** field.
- If no node matches the label, the Pod remains **Pending**.
- As soon as a matching node is labeled, Kubernetes schedules the Pod automatically.
- Use `kubectl label node` to add or remove node labels.
