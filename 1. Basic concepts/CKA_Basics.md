
# CKA Basics

> Consolidated notes for Certified Kubernetes Administrator (CKA)

## Downloads

- Ubuntu: https://ubuntu.com/download/desktop
- VMware Workstation Pro: https://www.vmware.com/info/workstation-pro/evaluation

---

# Docker & Kubernetes

- **Docker (Container Runtime Engine)** runs containers.
- **Kubernetes (K8s)** is an open-source container orchestration platform originally developed by Google and now maintained by the CNCF.

## Orchestra Analogy

| Orchestra | Kubernetes |
|-----------|------------|
| Musicians | Containers |
| Conductor | Kubernetes |
| Music Sheet | YAML Files |

Without a conductor there is confusion. With Kubernetes, containers work together as a symphony.

---

# Why Kubernetes?

- Cost optimization
- High Availability
- Disaster Recovery
- Stability
- Portability
- Scalability
- Resiliency
- Efficient CPU/Memory utilization
- Enterprise Security
  - RBAC
  - Network Policies
  - Secrets
  - Pod Security

Other orchestrators:
- Docker Swarm
- OpenShift
- Amazon ECS
- Kubernetes

---

# What is Kubernetes?

Kubernetes automates deployment, scaling and management of containerized applications.

Google donated Kubernetes to the **Cloud Native Computing Foundation (CNCF).**

---

# CNCF

Maintains cloud-native projects.

Examples:

| Project | Purpose |
|---------|---------|
| Kubernetes | Container Orchestration |
| Prometheus | Monitoring |
| Envoy | Service Mesh/API Gateway |
| Helm | Package Manager |
| containerd | Container Runtime |
| Fluentd | Log Aggregation |

---

# Container Orchestration

- Provisioning
- Deployment
- Scheduling
- Configuration
- Resource Allocation
- Load Balancing
- Monitoring
- Self Healing

---

# Physical vs VM vs Containers

## Physical Server
- Hardware
- OS
- Applications

## Virtual Machine
- Hypervisor
- Separate Guest OS
- Separate binaries

## Containers
- Share host kernel
- Lightweight
- Portable
- Fast startup

---

# Kubernetes Architecture

## Control Plane

- API Server
- Scheduler
- Controller Manager
- Cloud Controller Manager
- etcd

## Worker Node

- Kubelet
- kube-proxy
- Container Runtime
- Pods

---

# Components

## API Server
Front door to Kubernetes.

## etcd
Cluster database.

## Scheduler
Selects the best node.

## Controller Manager
Maintains desired state.

## Cloud Controller Manager
Cloud integration.

## Kubelet
Runs Pods on each node.

## kube-proxy
Networking and Services.

---

# Pod

A Pod is the smallest deployable object in Kubernetes.

Contains:
- One or more containers
- Shared networking
- Shared storage

---

# kubectl Flow

```
kubectl
    ↓
API Server
    ↓
etcd
    ↓
Scheduler
    ↓
Node
    ↓
Kubelet
    ↓
CRI
    ↓
Container Runtime
    ↓
Container
```

REST operations:
- GET
- POST
- PUT
- DELETE

Flow:

1. User submits request.
2. API Server validates.
3. Saved into etcd.
4. Scheduler selects node.
5. Kubelet starts Pod.
6. kube-proxy enables networking.
7. Controller Manager maintains state.

---

# Container Runtime Interface (CRI)

```
Kubelet
   ↓
CRI
   ↓
containerd / CRI-O
   ↓
Containers
```

CRI allows Kubernetes to support multiple runtimes.

---

# Kubernetes Installation Types

## KIND

- Kubernetes in Docker
- Development

## kubeadm

- Production cluster setup

## Minikube

- Single-node local cluster

---

# API Lifecycle

| Stage | Usage |
|------|------|
| Alpha | Experimental |
| Beta | Production candidate |
| Stable | Recommended |

---

# Basic Commands

## Versions

```bash
kubeadm version
```

## API Resources

```bash
kubectl api-resources
kubectl api-resources | grep pod
```

## Pods

```bash
kubectl get pods
kubectl get pods -o wide
kubectl describe pod poda
kubectl logs poda
kubectl exec -it poda -- /bin/bash
```

## Images

```bash
docker images
crictl images
```

## Create Pod

```bash
kubectl run poda --image=nginx:latest
```

## Nodes

```bash
kubectl get nodes
kubectl describe node <node>
```

---

# kubeconfig

Default:

Linux

```text
~/.kube/config
```

Windows

```text
C:\Users\<user>\.kube\config
```

Alternate:

```bash
kubectl get nodes --kubeconfig custom-kubeconfig
```

---

# Minikube

https://minikube.sigs.k8s.io/docs/start/

---

# Docker vs Kubernetes

| Docker | Kubernetes |
|---------|------------|
| docker run | kubectl run |
| docker ps | kubectl get pods |
| docker logs | kubectl logs |
| docker inspect | kubectl describe pod |
| docker exec | kubectl exec |
| docker rm | kubectl delete pod |

---

# Important Notes

- Pods always run on Nodes.
- Nodes are managed by the Control Plane.
- A Node can run multiple Pods.

Check Nodes

```bash
kubectl get nodes
kubectl describe node master01-control-plane
```
