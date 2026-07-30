# Kubernetes Architecture

Kubernetes follows a **Control Plane + Worker Node architecture**.

The Kubernetes cluster is divided into:

1. Control Plane (Master Nodes)
2. Worker Nodes

```
                         Kubernetes Cluster
                     -------------------------

                    Control Plane / Master Node

        +------------------------------------------------+
        |                                                  |
        |  kube-apiserver                                  |
        |  etcd                                            |
        |  kube-scheduler                                  |
        |  kube-controller-manager                         |
        |  cloud-controller-manager -----------------------+---> Cloud
        |                                                  |
        +--------------------------+-----------------------+
                                   |
                                   |
        -----------------------------------------------------
        |                          |                          |

   Worker Node 1              Worker Node 2              Worker Node 3

   kubelet                    kubelet                    kubelet
   kube-proxy                 kube-proxy                 kube-proxy
   container runtime          container runtime          container runtime

   Pods                       Pods                       Pods
```

The **Control Plane acts as the brain of Kubernetes**, responsible for managing the cluster state.

The **Worker Nodes execute application workloads** by running Pods.

---

## 1. Control Plane Components

| Component | Description |
|---|---|
| **kube-apiserver** | The front-end for the Kubernetes control plane. It exposes the Kubernetes API, which is used by other components and users to interact with the cluster. |
| **etcd** | Key-value store used to store cluster configuration data and state. |
| **kube-scheduler** | Selects the optimal node for each Pod to run on, based on resource requirements, constraints, and other factors. |
| **kube-controller-manager** | Each controller monitors the state of the cluster and makes changes to bring it to the desired state. |
| **cloud-controller-manager** | Integrates with cloud providers to manage cloud-specific resources like load balancers, storage, and virtual machines. |

> 📌 **Tip:** `kube-apiserver` is the only Control Plane component every other component talks to — `etcd`, `kube-scheduler`, `kube-controller-manager`, and every `kubelet` all go *through* it. Nothing talks directly to `etcd` except the API server.

```
                     ┌──────────────────────────────┐
                     │      Kubernetes Master        │
                     │                                │
                     │  kube-controller-manager   cloud-controller-manager ──▶ Cloud
                     │            │                        │
                     │            └───────┬────────────────┘
                     │                    ▼
                     │   etcd ◀────▶ kube-apiserver ◀───▶ kube-scheduler
                     └────────────────────┬───────────────┘
                                          │
                     ┌────────────────────┼───────────────────────┐
                     │              Worker Nodes                  │
                     │  kubelet        kubelet        kubelet     │
                     │  kube-proxy     kube-proxy     kube-proxy  │
                     │   (node 1)       (node 2)       (node 3)   │
                     └─────────────────────────────────────────────┘
```

---

## kube-apiserver

The API Server is the **front door of Kubernetes**. Every request made to the cluster goes through it.

Examples of clients that talk to the API Server:
- `kubectl` commands
- Kubernetes Dashboard
- CI/CD tools
- Terraform
- Controllers
- Scheduler
- Kubelets

### Request lifecycle

```bash
kubectl get pods
```
becomes a REST API request:
```
GET /api/v1/pods
```

Before processing any request, the API Server performs:
- Authentication
- Authorization
- Admission Controllers
- Request validation

After validation, the desired state is stored in **etcd**.

### If kube-apiserver is stopped
- `kubectl` commands will **NOT** work
- New changes cannot be submitted
- Existing containers continue running

```bash
kubectl get pods   # will fail
```

However:
```bash
crictl pods   # may still work — talks directly to the container runtime
```

> 📌 **Tip:** This is the fastest way to distinguish an API-server outage from a full cluster outage during troubleshooting — if `crictl`/`ctr` on the node still shows running containers but `kubectl` is unresponsive, the workloads are healthy and only the control plane's front door is down.

### Interview Question

**Q: Does `kubectl` communicate directly with `etcd`?**

**A:** No. All communication goes through `kube-apiserver`.

```
kubectl
  |
  v
kube-apiserver
  |
  v
etcd
```

---

## etcd

`etcd` is Kubernetes' distributed key-value database. It stores the **complete cluster state**.

Examples of what lives in etcd:
- Pods
- Nodes
- Secrets
- ConfigMaps
- Services
- Deployments
- Namespaces
- Cluster configuration

Think of etcd as: **the single source of truth for Kubernetes.**

If etcd is lost without a backup:
- Kubernetes cannot recover cluster state
- Cluster restoration is not possible

> 📌 **Tip:** This is the single most important reason to have a tested `etcdctl snapshot save`/restore process — an unrecoverable etcd means an unrecoverable cluster, regardless of how healthy your nodes and workloads are.

### If etcd is stopped
- `kubectl` commands may hang
- New objects cannot be created
- Existing workloads may continue running

```bash
kubectl get pods   # may not return
crictl ps          # may still show running containers
```

### Interview Question

**Q: Is etcd a relational database?**

**A:** No. etcd is a distributed key-value database.

---

## kube-scheduler

The Scheduler is responsible for assigning Pods to Worker Nodes.

It watches the API Server for **Pending Pods** (Pods without a node assignment) and selects the best node based on:
- CPU availability
- Memory availability
- Resource requests
- Taints and Tolerations
- Node Affinity
- Pod Affinity
- Labels
- Topology constraints

The Scheduler:
- Does **NOT** start containers
- Does **NOT** communicate with the container runtime

It only selects the node.

### If kube-scheduler is stopped

```bash
kubectl run nginx --image=nginx
```
Pod status: `Pending`

Once the Scheduler starts again: `Pending → Running` — the Pod gets assigned to a Worker Node.

> 📌 **Tip:** A Pod stuck in `Pending` isn't always a scheduler-down situation — check `kubectl describe pod` first, since insufficient resources, unmet node affinity/taints, or no matching nodes produce the exact same `Pending` state even with a perfectly healthy scheduler.

---

## kube-controller-manager

The Controller Manager continuously compares:

```
Desired State
     |
     v
Current State
```

If there's a difference, controllers take action to bring the cluster back to the desired state.

**Example:** Desired `replicas: 5`, current `running pods: 3` → Controller Manager creates 2 additional Pods.

### Controllers inside Controller Manager

| Controller | Function |
|---|---|
| Service Account Controller | Creates default ServiceAccounts for new namespaces |
| Replication Controller | Ensures the correct number of pod replicas are running for a ReplicationController or ReplicaSet |
| Deployment Controller | Manages Deployment objects by creating and updating ReplicaSets. Handles rollouts and rollbacks |
| Job Controller | Watches Job objects and creates Pods to run specific tasks to completion |
| Namespace Controller | Watches for Namespace deletion signals and deletes all resources contained within that namespace |
| Endpoint Controller | Populates Endpoints objects (which link Services to Pods) by watching Services and Pods |

The Controller Manager continuously watches the cluster and takes corrective action — this reconciliation loop (**observe → diff → act**) is the core pattern behind almost everything in Kubernetes, not just this component.

---

## cloud-controller-manager

Used when Kubernetes runs on a cloud platform (AWS, Azure, Google Cloud, etc.).

Responsibilities:
- Create cloud Load Balancers
- Manage Persistent Disks
- Configure cloud Routes
- Manage Node lifecycle

**Example:** Creating a `Service` of `Type: LoadBalancer` causes the cloud-controller-manager to request an actual cloud load balancer.

Without CCM: Kubernetes cannot automatically create cloud resources — this is exactly why on bare-metal/homelab clusters, `LoadBalancer` Services sit at `EXTERNAL-IP: <pending>` forever unless something like **MetalLB** is installed to fill that gap.

---

## Control Plane Binary Location

Kubernetes components are typically installed as binaries at:

```
/usr/local/bin
```

Examples:
```
kube-apiserver
kube-scheduler
kube-controller-manager
kubelet
kube-proxy
```

---

## 2. Worker Node Components

Worker Nodes run application workloads.

| Component | Description |
|---|---|
| **kubelet** | An agent that runs on each node and ensures that containers are running |
| **kube-proxy** | Maintains network rules on nodes to enable network communication to Pods |
| **Container Runtime** | Software responsible for running containers. Examples include containerd and CRI-O |
| **Pods** | Application workloads |

---

## kubelet

`kubelet` is an agent running on every Worker Node.

Responsibilities:
- Watches the API Server
- Receives Pod assignments
- Pulls container images
- Starts containers through the CRI
- Monitors container health
- Reports node and Pod status

### Pod startup process

```
API Server
     |
Scheduler assigns Pod
     |
kubelet detects assignment
     |
CRI request
     |
container runtime
     |
Container starts
```

**Important:** `kubelet` does **NOT** schedule Pods — that's the Scheduler's job. `kubelet` only executes what's already been assigned to its node.

---

## kube-proxy

`kube-proxy` manages networking on Worker Nodes.

Responsibilities:
- Service IP handling
- ClusterIP networking
- NodePort networking
- Load balancing
- iptables/IPVS rules

**Example traffic flow:**
```
Client
   |
Service IP
   |
kube-proxy
   |
Pod
```
`kube-proxy` forwards traffic to healthy Pods.

---

## Container Runtime

The Container Runtime is responsible for actually running containers.

Examples: **containerd**, **CRI-O**

Responsibilities:
- Pull container images
- Create containers
- Start containers
- Stop containers
- Manage container lifecycle

Kubernetes communicates with the runtime using the **CRI (Container Runtime Interface)** — this abstraction is what lets Kubernetes swap runtimes without changing kubelet code.

---

## Complete Pod Creation Flow

```
kubectl run nginx
     |
kube-apiserver
     |
Authentication
Authorization
Admission Controllers
     |
etcd stores desired state
     |
Scheduler detects Pending Pod
     |
Scheduler selects Worker Node
     |
API Server updated
     |
kubelet detects assigned Pod
     |
CRI
     |
container runtime
     |
Image pulled
     |
Container started
     |
kube-proxy provides networking
     |
Controller Manager monitors health
```

---

## Summary

| Area | Component | Responsibility |
|---|---|---|
| Control Plane | kube-apiserver | Kubernetes API entry point |
| Control Plane | etcd | Stores cluster state |
| Control Plane | kube-scheduler | Assigns Pods to Nodes |
| Control Plane | kube-controller-manager | Maintains desired state |
| Control Plane | cloud-controller-manager | Cloud integration |
| Worker Node | kubelet | Runs Pods |
| Worker Node | kube-proxy | Networking |
| Worker Node | Container Runtime | Runs Containers |

Kubernetes follows a **declarative model**:

```
User defines desired state
          |
          v
Kubernetes continuously works
          |
          v
Actual state matches desired state
```

---

## Additional Tips & Gotchas

- **Single point of truth, single point of failure:** because every component talks through `kube-apiserver` → `etcd`, a healthy control plane in a homelab is really about keeping these two processes up; everything else (scheduler, controller-manager) can restart and simply resume watching where it left off.
- **HA control planes:** in production, you run multiple `kube-apiserver` instances behind a load balancer and an odd-numbered etcd cluster (3 or 5 members) for quorum — a single-control-plane-node lab setup (like a typical kubeadm homelab) has none of this redundancy, which is worth calling out explicitly if asked about production readiness in an interview.
- **Static pods vs. control plane components:** on kubeadm clusters specifically, `kube-apiserver`, `etcd`, `kube-scheduler`, and `kube-controller-manager` usually run as **static Pods** managed directly by `kubelet` from manifests in `/etc/kubernetes/manifests/`, not as regular Deployments — this is exactly the kind of static-pod recovery scenario relevant to your homelab DHCP/IP-change troubleshooting.
- **kube-proxy modes:** `iptables` mode is the long-standing default; `IPVS` mode scales better for clusters with very large numbers of Services since it uses hash tables instead of sequential rule matching.
- **CRI matters for troubleshooting:** since kubelet only talks CRI, tools like `crictl` (not `docker`) are the correct way to inspect containers directly on a node when the API server is unreachable — useful with containerd, which doesn't expose a Docker-compatible socket by default.
