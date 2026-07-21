# Kubernetes Architecture

Kubernetes follows a **Control Plane + Worker Node** architecture.

```
                   Kubernetes Cluster
                 ----------------------
                |     Control Plane     |
                |-----------------------|
                | API Server            |
                | Scheduler             |
                | Controller Manager    |
                | Cloud Controller Mgr  |
                | etcd                  |
                 -----------------------
                     |            |
        ---------------------------------------
        |                                     |
   Worker Node 1                        Worker Node 2
   ----------------                    ----------------
   Kubelet                             Kubelet
   kube-proxy                          kube-proxy
   CRI (containerd)                    CRI (containerd)
   Pods                                Pods
```

The Control Plane acts as the **brain** of Kubernetes, while Worker Nodes execute the workloads.

---

# Kubernetes Components

## 1. API Server

The **API Server** is the front door of Kubernetes.

Every request made to the cluster goes through the API Server.

Examples:

- kubectl commands
- Kubernetes Dashboard
- CI/CD tools
- Terraform
- Controllers
- Scheduler
- Kubelets

All communicate with the API Server.

Example:

```bash
kubectl get pods
```

The above command becomes a REST API call:

```
GET /api/v1/pods
```

The API Server validates:

- Authentication
- Authorization
- Admission Controllers
- Request validation

Once validated, it stores the desired state in **etcd**.

### Interview Question

**Q:** Does kubectl communicate directly with etcd?

**Answer:** No. Every request always goes through the API Server.

---

## 2. etcd

etcd is Kubernetes' distributed key-value database.

It stores:

- Pods
- Nodes
- Secrets
- ConfigMaps
- Services
- Deployments
- Namespaces
- Cluster configuration

Think of etcd as the **single source of truth**.

If etcd is lost without a backup, the cluster cannot recover.

### Interview Question

**Q:** Is etcd relational?

No.

It is a distributed key-value database.

---

## 3. Scheduler

The Scheduler watches the API Server for Pods that do not yet have a Node assigned.

It decides where the Pod should run based on:

- CPU
- Memory
- Taints & Tolerations
- Node Affinity
- Pod Affinity
- Resource requests
- Labels
- Topology

The Scheduler **does not run containers**.

It only selects the best node.

After making its decision, it updates the API Server.

---

## 4. Controller Manager

The Controller Manager constantly compares:

```
Desired State
        vs
Current State
```

Example:

Desired replicas = 5

Current replicas = 3

Controller Manager automatically creates two more Pods.

Controllers include:

- ReplicaSet Controller
- Deployment Controller
- Job Controller
- Namespace Controller
- Node Controller

It continuously watches the cluster.

---

## 5. Cloud Controller Manager

Used only on cloud providers.

Examples:

AWS
Azure
Google Cloud

Responsibilities:

- Create Load Balancers
- Manage Persistent Disks
- Configure Routes
- Node lifecycle

Without CCM, Kubernetes cannot automatically provision cloud resources.

---

## 6. Kubelet

Kubelet is installed on every Worker Node.

It watches the API Server.

If the Scheduler assigns a Pod to that Node:

1. Pull image
2. Ask CRI to start container
3. Monitor health
4. Report status

Kubelet never schedules Pods.

---

## 7. kube-proxy

Responsible for networking.

Functions:

- Service IP
- ClusterIP
- NodePort
- Load balancing
- iptables/IPVS rules

When traffic reaches a Service,
kube-proxy forwards it to one of the healthy Pods.

---

## 8. Worker Node

A Worker Node contains:

- Kubelet
- kube-proxy
- Container Runtime
- Pods

Worker Nodes execute application workloads.

---

# Complete Pod Creation Flow

```
kubectl run nginx
        |
        v
API Server
        |
        v
Authentication
Authorization
Admission Controllers
        |
        v
etcd stores desired state
        |
        v
Scheduler detects Pending Pod
        |
        v
Selects Best Worker Node
        |
        v
Writes assignment to API Server
        |
        v
Kubelet notices Pod assigned
        |
        v
CRI
        |
        v
containerd
        |
        v
Image Pulled
        |
        v
Container Started
        |
        v
kube-proxy exposes networking
        |
        v
Controller Manager continuously monitors health
```
