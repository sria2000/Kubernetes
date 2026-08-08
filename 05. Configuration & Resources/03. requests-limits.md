# Kubernetes Resource Requests and Limits

## Official Documentation

https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/

---

# Why Do We Need Resource Requests and Limits?

In a Kubernetes cluster, multiple Pods share the CPU and memory available on worker nodes.

Without resource control, one Pod can consume excessive CPU or memory, affecting other applications running on the same node.

## Challenges

### Challenge 1 - Resource Starvation

A Pod may consume more CPU or memory than expected.

As a result:

- Other Pods become slow.
- Applications may stop responding.
- The node can run out of memory.

---

### Challenge 2 - Guaranteed Resources

Some applications require a minimum amount of CPU and memory to function correctly.

Examples:

- Database servers
- Java applications
- Kafka
- Elasticsearch

If these applications do not receive sufficient resources, they may perform poorly or fail to start.

---

# What are Requests and Limits?

Kubernetes allows us to control resource usage using **Requests** and **Limits**.

Resources commonly controlled:

- CPU
- Memory

Example:

```
Memory Request = 256Mi
Memory Limit   = 384Mi
```

This means:

- Kubernetes guarantees at least **256Mi** memory.
- The container cannot use more than **384Mi** memory.

---

# Difference Between Request and Limit

| Requests | Limits |
|----------|---------|
| Minimum amount of CPU or memory guaranteed to a container. | Maximum amount of CPU or memory a container can consume. |
| Used by the scheduler while placing Pods on nodes. | Prevents a container from consuming excessive resources. |
| Ensures enough resources are available before scheduling. | CPU is throttled, and excessive memory usage may cause the container to be terminated (OOMKilled). |

---

# How Kubernetes Uses Requests and Limits

```
          Create Pod
               │
               ▼
      Scheduler checks Requests
               │
               ▼
Node has enough CPU & Memory?
        │               │
       Yes              No
        │               │
        ▼               ▼
 Schedule Pod      Pod remains Pending
        │
        ▼
 Container Starts
        │
        ▼
 Can use resources up to Limit
        │
        ▼
 Exceeds Limit?
        │
   ┌────┴────┐
   │         │
  CPU      Memory
Throttle   Container Killed
```

---

# Base Pod Manifest

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: srilabs-pod

spec:
  containers:
  - name: srilabs-container
    image: nginx
```

---

# Pod Manifest with Requests and Limits

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: srilabs-pod

spec:
  containers:
  - name: srilabs-container
    image: nginx

    resources:
      requests:
        memory: "128Mi"
        cpu: "100m"

      limits:
        memory: "500Mi"
        cpu: "1"
```

Deploy:

```bash
kubectl apply -f request-limit.yaml
```

---

# Understanding Memory

```yaml
requests:
  memory: "128Mi"
```

Meaning:

- The container requests **128 MiB** of memory.
- Kubernetes schedules the Pod only on a node that has at least **128Mi** available.

---

```yaml
limits:
  memory: "500Mi"
```

Meaning:

- Maximum memory the container can consume.
- If the application exceeds **500Mi**, Linux may terminate the container with an **OOMKilled (Out of Memory)** event.

---

# Understanding CPU

```yaml
requests:
  cpu: "100m"
```

or

```yaml
cpu: "0.1"
```

Both are equivalent.

Meaning:

- Requests **100 millicores** (10% of one CPU core).
- Kubernetes guarantees this amount while scheduling.

Examples:

| CPU Value | Meaning |
|-----------|---------|
| 100m | 0.1 CPU Core |
| 250m | 0.25 CPU Core |
| 500m | 0.5 CPU Core |
| 1000m | 1 Full CPU Core |

---

```yaml
limits:
  cpu: "1"
```

Meaning:

- Maximum CPU usage = **1 CPU Core**.
- If the application tries to consume more CPU, Kubernetes throttles it instead of killing it.

---

# Requests vs Limits Example

Suppose a worker node has:

```
CPU    : 4 cores
Memory : 8 GB
```

A Pod requests:

```
CPU Request    = 500m
Memory Request = 512Mi
```

Scheduler checks whether the node has:

- 0.5 CPU available
- 512Mi memory available

If yes:

```
Pod is scheduled.
```

The container can later consume resources up to its configured limits.

---

# Checking Node Capacity

```bash
kubectl describe node node01
```

Example:

```
Capacity:

cpu:      1
memory:   1948916Ki

Allocatable:

cpu:      1
memory:   1846516Ki
```

## Capacity

Total hardware resources available on the worker node.

---

## Allocatable

Resources available for Pods after reserving resources for:

- Kubernetes components
- Operating System
- Kubelet
- System processes

---

# Viewing Pod Resource Usage on a Node

Near the bottom of:

```bash
kubectl describe node node01
```

You'll see:

```
Non-terminated Pods

Namespace      Name          CPU Requests  CPU Limits
default        srilabs-pod   100m (10%)    1 (100%)
```

Memory:

```
Memory Requests : 128Mi
Memory Limits   : 500Mi
```

This shows how much CPU and memory each Pod has requested and the limits configured.

---

# Allocated Resources

Example:

```
Allocated resources:

CPU Requests : 200m (20%)
CPU Limits   : 2 (200%)

Memory Requests : 138Mi (7%)
Memory Limits   : 1524Mi (84%)
```

Notice:

```
CPU Limits = 200%
```

This is possible because Kubernetes supports **CPU overcommit**.

Multiple Pods can request more CPU than physically available because not every application uses its maximum CPU all the time.

---

# Useful Commands

Deploy Pod

```bash
kubectl apply -f request-limit.yaml
```

View Pods

```bash
kubectl get pods
```

Describe Pod

```bash
kubectl describe pod srilabs-pod
```

Describe Node

```bash
kubectl describe node node01
```

Delete Pod

```bash
kubectl delete -f request-limit.yaml
```

---

# Advantages of Requests and Limits

| Advantage | Description |
|-----------|-------------|
| Prevents resource starvation | Prevents one container from consuming all CPU or memory on a node. |
| Fair resource allocation | Ensures resources are shared among all Pods. |
| Better scheduling | Scheduler places Pods only on nodes with sufficient resources. |
| Prevents over-provisioning | Limits excessive resource consumption. |
| Improved stability | Reduces application failures due to resource contention. |
| Better performance | Critical applications receive guaranteed resources. |

---

# Key Points to Remember

- **Requests** guarantee the minimum CPU and memory required by a container.
- **Limits** define the maximum CPU and memory a container can use.
- Kubernetes uses **Requests** during scheduling.
- Exceeding the **CPU limit** results in CPU throttling.
- Exceeding the **Memory limit** may cause the container to be terminated (**OOMKilled**).
- Requests help Kubernetes choose an appropriate worker node.
- Properly configured Requests and Limits improve cluster stability, fairness, and application performance.
