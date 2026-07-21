# Troubleshooting: `kubectl` Error on Worker Node After Joining the Cluster

## Problem

After successfully joining the worker node (`k8-node01`) to the Kubernetes cluster, running `kubectl` on the worker resulted in the following error:

```bash
kubectl get pods
```

Output:

```text
E0719 ... couldn't get current server API group list:
Get "http://localhost:8080/api?timeout=32s":
dial tcp 127.0.0.1:8080: connect: connection refused

The connection to the server localhost:8080 was refused - did you specify the right host or port?
```

---

# Why does this happen?

This error **does not mean the worker node failed to join the cluster.**

`kubectl` requires a **kubeconfig** file to know how to connect to the Kubernetes API Server.

Since no kubeconfig exists on the worker node, `kubectl` defaults to connecting to:

```
http://localhost:8080
```

There is **no API Server** running on the worker node, so the connection is refused.

---

# Step 1 - Verify the Worker Joined Successfully

Run the following on the **Master Node**:

```bash
kubectl get nodes
```

Example:

```text
NAME         STATUS   ROLES           AGE    VERSION
k8-node01    Ready    <none>          82m    v1.35.6
k8s-master   Ready    control-plane   158m   v1.35.6
```

If the worker node appears with **STATUS = Ready**, then the node has joined the cluster successfully.

---

# Step 2 - Verify the kubelet Service

On the **Worker Node**:

```bash
systemctl status kubelet
```

Expected:

```text
Active: active (running)
```

This confirms the kubelet is healthy and communicating with the control plane.

---

# Step 3 - Verify kubelet Configuration

Ensure the worker has its kubelet configuration.

```bash
cat /etc/kubernetes/kubelet.conf
```

If this file exists, the node has successfully joined the cluster.

---

# Step 4 - Copy the Admin kubeconfig (Optional)

> **Note**
>
> This step is only required if you want to run `kubectl` commands directly from the worker node.
>
> In production environments, `kubectl` is typically executed from the Control Plane (Master) or from a dedicated administration workstation.

Copy the Kubernetes admin configuration from the Master to the Worker.

If SSH as **root** is enabled:

```bash
scp /etc/kubernetes/admin.conf root@k8-node01:/root/.kube/config
```

If root SSH is disabled, copy the file using a normal user (for example, `sri`) and then move it into `/root/.kube/config` using `sudo`.

Example:

```bash
scp /etc/kubernetes/admin.conf sri@k8-node01:/home/sri/
```

On the worker:

```bash
sudo mkdir -p /root/.kube
sudo mv /home/sri/admin.conf /root/.kube/config
```

---

# Step 5 - Configure kubectl

On the Worker:

```bash
export KUBECONFIG=/root/.kube/config
```

To make this permanent:

```bash
echo 'export KUBECONFIG=/root/.kube/config' >> ~/.bashrc
source ~/.bashrc
```

---

# Step 6 - Verify kubectl

Now run:

```bash
kubectl get nodes
```

Expected:

```text
NAME         STATUS   ROLES           AGE    VERSION
k8-node01    Ready    <none>          88m    v1.35.6
k8s-master   Ready    control-plane   164m   v1.35.6
```

Verify Pods:

```bash
kubectl get pods
```

Example:

```text
NAME   READY   STATUS    RESTARTS   AGE
poda   1/1     Running   0          112m
```

---

# Root Cause

The worker node had successfully joined the Kubernetes cluster.

However, `kubectl` was unable to communicate with the API Server because it did not have a valid **kubeconfig** file.

Without a kubeconfig, `kubectl` attempts to connect to:

```
http://localhost:8080
```

Since the Kubernetes API Server runs only on the Control Plane, the connection is refused.

After copying `admin.conf` and setting the `KUBECONFIG` environment variable, `kubectl` successfully connected to the Control Plane and all commands worked as expected.

---

# Key Takeaways

- ✅ The worker node had joined the cluster successfully.
- ✅ `kubelet` was healthy and running.
- ✅ The error was caused by a missing kubeconfig file.
- ✅ Copying `admin.conf` and setting `KUBECONFIG` resolved the issue.
- ✅ In production, it is recommended to run `kubectl` from the Control Plane or a dedicated administration machine rather than from worker nodes.
