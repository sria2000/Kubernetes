# Kind Kubernetes Cluster Installation on Ubuntu

## Overview

**Kind (Kubernetes IN Docker)** is a tool used to run local Kubernetes clusters using Docker container nodes.

It is mainly used for:

- Kubernetes learning
- Testing applications
- CI/CD pipelines
- Multi-node Kubernetes cluster testing

This guide covers:

- Installing Docker
- Installing Kind
- Installing kubectl
- Creating a single-node Kubernetes cluster
- Creating a multi-node Kubernetes cluster
- Managing Kind clusters


---

# 1. Open Killercoda Ubuntu Session

Open a Ubuntu environment in Killercoda.

Before installing Kind, ensure Docker is available.


---

# 2. Prerequisites

## Check Docker Installation

Verify Docker is installed and running.

```bash
docker version
```

Check Docker service:

```bash
systemctl status docker
```

If Docker is not installed, follow the installation steps below.


---

# 3. Install Required Packages

Update Ubuntu packages:

```bash
sudo apt update
sudo apt upgrade -y
```

Restart services:

```bash
sudo systemctl restart ssh.service

sudo systemctl restart unattended-upgrades.service
```

Install Docker:

```bash
sudo apt install -y docker.io
```

Enable and start Docker:

```bash
sudo systemctl enable --now docker
```

Verify Docker:

```bash
docker --version
```


Example:

```
Docker version 28.x.x
```


---

# 4. Install Kind

Download the latest Kind binary:

```bash
curl -Lo ./kind https://kind.sigs.k8s.io/dl/latest/kind-linux-amd64
```

Make it executable:

```bash
chmod +x ./kind
```

Move Kind binary into PATH:

```bash
sudo mv ./kind /usr/local/bin/kind
```

Verify installation:

```bash
kind version
```


Example:

```
kind v0.xx.x
```


---

# 5. Verify Docker Images

Check available Docker images:

```bash
docker images
```


---

# 6. Create a Single Node Kubernetes Cluster

Create a default Kind cluster:

```bash
kind create cluster --name sri-kind-cluster
```

Check available clusters:

```bash
kind get clusters
```


Example:

```
sri-kind-cluster
```


---

# 7. Install kubectl

Install required packages:

```bash
sudo apt-get install -y apt-transport-https ca-certificates curl gpg
```

Create Kubernetes key directory:

```bash
sudo install -p -m 0755 -d /etc/apt/keyrings
```

Download Kubernetes signing key:

```bash
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.35/deb/Release.key | \
sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```

Set permissions:

```bash
sudo chmod a+r /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```

Add Kubernetes repository:

```bash
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] \
https://pkgs.k8s.io/core:/stable:/v1.35/deb/ /' | \
sudo tee /etc/apt/sources.list.d/kubernetes.list
```

Install kubectl:

```bash
sudo apt update

sudo apt install -y kubectl
```


Verify:

```bash
kubectl version --client
```


---

# 8. Verify Kubernetes Cluster

Check cluster information:

```bash
kubectl cluster-info
```

Check nodes:

```bash
kubectl get nodes
```


Example:

```
NAME                         STATUS   ROLE
sri-kind-cluster-control-plane   Ready    control-plane
```


---

# 9. Install Kind on Windows Using Docker Desktop

## Prerequisites

Install:

- Docker Desktop
- Kind
- kubectl


Kind documentation:

https://kind.sigs.k8s.io/docs/user/quick-start/


Add Kind executable location to Windows PATH.


---

# 10. Create Kind Cluster on Windows

Example:

```powershell
kind create cluster
```


Output:

```
Creating cluster "kind" ...

✓ Ensuring node image
✓ Preparing nodes
✓ Writing configuration
✓ Starting control-plane
✓ Installing CNI
✓ Installing StorageClass

Set kubectl context to "kind-kind"
```


Verify:

```powershell
kind get clusters
```


Output:

```
kind
```


---

# 11. Create Multi Node Kubernetes Cluster

By default Kind creates only one control-plane node.

For multi-node testing, create a configuration file.


Create:

```
kind-config.yaml
```


Add the following:


```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4

nodes:
- role: control-plane
- role: worker
- role: worker
```


---

# 12. Create Multi Node Kind Cluster


Run:

```bash
kind create cluster \
--name sri-cluster \
--config kind-config.yaml
```


Example output:

```
Creating cluster "sri-cluster"

✓ Ensuring node image
✓ Preparing nodes
✓ Writing configuration
✓ Starting control-plane
✓ Installing CNI
✓ Installing StorageClass
✓ Joining worker nodes

Set kubectl context to "kind-sri-cluster"
```


---

# 13. Verify Multi Node Cluster


Check clusters:

```bash
kind get clusters
```


Example:

```
sri-cluster
```


Check Kubernetes nodes:

```bash
kubectl get nodes
```


Example output:

```
NAME                         STATUS   ROLES
sri-cluster-control-plane    Ready    control-plane
sri-cluster-worker           Ready    <none>
sri-cluster-worker2          Ready    <none>
```


---

# 14. Check Kubernetes Context


Display available contexts:

```bash
kubectl config get-contexts
```


Example:

```
CURRENT   NAME
*         kind-sri-cluster
```


The `*` indicates the active Kubernetes context.


---

# 15. Delete Kind Cluster


Delete a specific cluster:

```bash
kind delete cluster --name sri-cluster
```


Delete default cluster:

```bash
kind delete cluster --name kind
```


Verify:

```bash
kind get clusters
```


---

# 16. Useful Kind Commands


## List Clusters

```bash
kind get clusters
```


## Create Cluster

```bash
kind create cluster
```


## Create Cluster With Config

```bash
kind create cluster --config kind-config.yaml
```


## Delete Cluster

```bash
kind delete cluster --name <cluster-name>
```


## Kubernetes Nodes

```bash
kubectl get nodes
```


## Kubernetes Context

```bash
kubectl config get-contexts
```


---

# 17. Notes

- Kind uses Docker containers as Kubernetes nodes.
- Docker must be running before creating a Kind cluster.
- kubectl communicates with Kubernetes API server using kubeconfig.
- Docker Desktop installation automatically installs kubectl in many environments.
- Multi-node clusters require a Kind configuration YAML file.


---

# References

Kind Documentation:

https://kind.sigs.k8s.io/docs/user/quick-start/

Kubernetes Documentation:

https://kubernetes.io/docs/
