# Kubernetes Worker Nodes

Worker nodes are the machines where your application Pods actually run. A Kubernetes cluster can have one or many worker nodes depending on the workload.

---

# Why Do We Need Multiple Worker Nodes?

## 1. Distribute Workloads

Having multiple worker nodes allows Kubernetes to spread Pods across different machines instead of placing everything on a single server.

Example:

- Worker Node 1 → Pod A
- Worker Node 2 → Pod B
- Worker Node 3 → Pod C

This improves resource utilization and scalability.

---

## 2. Add Nodes Anytime

One of Kubernetes' biggest advantages is that you can increase cluster capacity whenever required.

Example:

Initially:

- 2 Worker Nodes

Later:

- Add a 3rd Worker Node
- Kubernetes automatically starts scheduling new Pods on the new node.

No application changes are required.

---

## 3. Fault Tolerance (High Availability)

Multiple worker nodes improve the availability of your applications.

If one worker node goes down:

- Pods running on that node become unavailable.
- Kubernetes recreates those Pods on healthy worker nodes (provided sufficient resources are available).
- The application continues running with minimal disruption.

This is one of Kubernetes' self-healing features.

---

## 4. Different Node Sizes

Not all worker nodes need to have the same hardware specifications.

Examples:

### Development Cluster

- 2 CPU
- 4 GB RAM

Suitable for testing and development workloads.

### Production Cluster

- 16 CPU
- 64 GB RAM

Suitable for production applications that require higher performance.

Kubernetes schedules Pods based on available resources and scheduling constraints.

---

# List Worker Nodes

Display all nodes in the cluster:

```bash
kubectl get nodes
```

Example output:

```text
NAME      STATUS   ROLES           AGE   VERSION
master    Ready    control-plane   10d   v1.36.0
worker1   Ready    <none>          10d   v1.36.0
worker2   Ready    <none>          10d   v1.36.0
```

---

# Create a Deployment

Create a Deployment with 3 replicas:

```bash
kubectl create deployment nginx-deployment \
  --image=nginx \
  --replicas=3
```

---

# View Where Pods Are Running

List Pods along with the worker node where each Pod is scheduled:

```bash
kubectl get pods -o wide
```

Example:

```text
NAME                                READY   STATUS    NODE
nginx-deployment-7d8f8c4d4-abc12    1/1     Running   worker1
nginx-deployment-7d8f8c4d4-def34    1/1     Running   worker2
nginx-deployment-7d8f8c4d4-ghi56    1/1     Running   worker1
```

The **NODE** column shows which worker node each Pod is running on.

---

# Removing a Worker Node

When a worker node is cordoned (disabled for scheduling):

```bash
kubectl cordon worker1
```

Checking the nodes:

```bash
kubectl get nodes
```

Example:

```text
NAME      STATUS
worker1   Ready,SchedulingDisabled
worker2   Ready
```

### What does `SchedulingDisabled` mean?

- The node is still running existing Pods.
- Kubernetes **does not schedule any new Pods** on that node.
- New Pods are scheduled only on nodes whose status is **Ready**.

To completely move existing Pods off the node, use:

```bash
kubectl drain worker1 --ignore-daemonsets
```

To allow scheduling again:

```bash
kubectl uncordon worker1
```

---

# Summary

- Worker nodes run your application Pods.
- Multiple worker nodes distribute workloads efficiently.
- New worker nodes can be added at any time to increase capacity.
- Multiple nodes provide fault tolerance and high availability.
- Worker nodes can have different CPU, memory, and hardware capacities.
- Use `kubectl get nodes` to list cluster nodes.
- Use `kubectl get pods -o wide` to see which worker node each Pod is running on.
- A node marked as **SchedulingDisabled** will not receive new Pods, while **Ready** nodes continue to accept new workloads.
