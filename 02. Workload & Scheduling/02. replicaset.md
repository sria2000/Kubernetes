# ReplicaSet

Docs: https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/

---

## 1. Basics — Manifest, Create, Inspect, Delete

### Manifest

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: frontend
  labels:
    app: guestbook
    tier: frontend
spec:
  # modify replicas according to your case
  replicas: 3
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
      - name: php-redis
        image: us-docker.pkg.dev/google-samples/containers/gke/gb-frontend:v5
```

Create the ReplicaSet:

```bash
k apply -f replica.yaml
# replicaset.apps/frontend created
```

### Check Pods

```bash
k get pods
# NAME             READY   STATUS              RESTARTS   AGE
# frontend-dvmpv   0/1     ContainerCreating   0          19s
# frontend-wwk7x   0/1     ContainerCreating   0          19s
# frontend-xtsqj   0/1     ContainerCreating   0          19s
```

Once running:

```bash
k get pods
# NAME             READY   STATUS    RESTARTS   AGE
# frontend-dvmpv   1/1     Running   0          59s
# frontend-wwk7x   1/1     Running   0          59s
# frontend-xtsqj   1/1     Running   0          59s
```

Three pods are created because `replicas: 3` — each gets a random suffix appended to the ReplicaSet name.

### Check the ReplicaSet

```bash
k get rs
# NAME       DESIRED   CURRENT   READY   AGE
# frontend   3         3         3       63s
```

### Describe the ReplicaSet

```bash
k describe rs frontend
```

Key details returned:

- **Selector:** `tier=frontend` — how the ReplicaSet knows which pods it owns
- **Replicas:** 3 current / 3 desired
- **Pods Status:** 3 Running / 0 Waiting / 0 Succeeded / 0 Failed
- **Pod Template:** the container spec (`php-redis` image) used to create each pod

Events show a `SuccessfulCreate` entry for each of the three pods.

### Info on One Pod

```bash
k get pods frontend-dvmpv -o yaml
```

Check labels on all pods:

```bash
kubectl get pods --show-labels
# NAME             READY   STATUS    RESTARTS   AGE   LABELS
# frontend-dvmpv   1/1     Running   0          14m   tier=frontend
# frontend-wwk7x   1/1     Running   0          14m   tier=frontend
# frontend-xtsqj   1/1     Running   0          14m   tier=frontend
```

Check labels on a single pod:

```bash
kubectl describe pod frontend-dvmpv | grep -i labe
# Labels:           tier=frontend
```

### Delete the ReplicaSet

```bash
k get rs
# NAME       DESIRED   CURRENT   READY   AGE
# frontend   3         3         3       17m

k delete rs frontend
# replicaset.apps "frontend" deleted from default namespace

k get rs
# No resources found in default namespace.
```

Deleting the ReplicaSet also deletes the pods it owns.

---

## 2. Labels, Selectors, and Naming Convention

One of the most confusing topics for Kubernetes beginners is understanding the different names used in a Deployment or ReplicaSet.

### Real World Example

Imagine **TCS** is an IT company.

- **Deployment** = HR Manager
- **ReplicaSet** = Team Leader
- **Pods** = Employees
- **Container** = Laptop assigned to each employee
- **Image** = Standard Operating System (nginx)
- **Label** = Employee's Department

```
                    TCS Company
                         |
                  Deployment (HR)
                         |
                 ReplicaSet (Team Lead)
                         |
        +----------------+----------------+
        |                |                |
      Employee1       Employee2       Employee3
        (Pod)           (Pod)           (Pod)
           |               |               |
      employee-laptop employee-laptop employee-laptop
           |               |               |
          nginx           nginx           nginx
```

### Example Deployment YAML

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: tcs-deployment

spec:
  replicas: 3

  selector:
    matchLabels:
      department: web-team

  template:
    metadata:
      labels:
        department: web-team

    spec:
      containers:
      - name: employee-laptop
        image: nginx
        ports:
        - containerPort: 80
```

### Understanding Every Name

**1. Deployment Name**

```yaml
metadata:
  name: tcs-deployment
```

The name of the Deployment object. View with `kubectl get deployments`.

**2. ReplicaSet Name**

You never create this manually — the Deployment automatically creates it, e.g. `tcs-deployment-5776bb48d4`. View with `kubectl get rs`.

**3. Pod Names**

Pods are automatically created by the ReplicaSet, e.g.:

```
tcs-deployment-5776bb48d4-nmk8r
tcs-deployment-5776bb48d4-vbmgl
tcs-deployment-5776bb48d4-zttwb
```

View with `kubectl get pods`. Naming convention:

```
Deployment Name
        |
        +-----------------------+
                                |
                       ReplicaSet Hash
                                |
                          Random Suffix
```

**4. Container Name**

```yaml
containers:
- name: employee-laptop
```

View with `kubectl describe pod <pod-name>`. This is **NOT** the Pod name.

**5. Image**

```yaml
image: nginx
```

The Docker image that Kubernetes downloads to run inside the container.

### What is a Label?

A label is a key-value pair attached to an object.

```yaml
labels:
  department: web-team
```

Think of it as an employee's department. Every Pod created by this Deployment gets this label.

### Selector

The Deployment says: *"Manage every Pod whose department is web-team."*

```yaml
selector:
  matchLabels:
    department: web-team
```

The Pods receive:

```yaml
labels:
  department: web-team
```

Because they match, the Deployment can manage them.

```
Deployment

Selector

department=web-team
        |
        |
        +----------------------+
        |          |           |
      Pod1       Pod2       Pod3
department=   department=  department=
web-team      web-team     web-team
```

### Viewing Labels

Show labels on every pod:

```bash
kubectl get pods --show-labels
# NAME                                LABELS
# tcs-deployment-5776bb48d4-nmk8r     department=web-team,pod-template-hash=5776bb48d4
# tcs-deployment-5776bb48d4-vbmgl     department=web-team,pod-template-hash=5776bb48d4
```

### Find Pods by Label

```bash
kubectl get pods -l department=web-team
# tcs-deployment-5776bb48d4-nmk8r
# tcs-deployment-5776bb48d4-vbmgl
# tcs-deployment-5776bb48d4-zttwb
```

### Describe a Pod

```bash
kubectl describe pod tcs-deployment-5776bb48d4-nmk8r
# Labels:
#     department=web-team
```

This confirms that the Pod has the label assigned by the Deployment.

### Relationship Between Components

```
Deployment
Name:
tcs-deployment

        |
        v

ReplicaSet
Name:
tcs-deployment-5776bb48d4

        |
        v

Pods

tcs-deployment-5776bb48d4-nmk8r    Label: department=web-team
tcs-deployment-5776bb48d4-vbmgl    Label: department=web-team
tcs-deployment-5776bb48d4-zttwb    Label: department=web-team
```

### Summary Table

| Kubernetes Object | Example | Purpose |
|-------------------|---------|---------|
| Deployment | `tcs-deployment` | Manages application lifecycle |
| ReplicaSet | `tcs-deployment-5776bb48d4` | Maintains desired number of Pods |
| Pod | `tcs-deployment-5776bb48d4-nmk8r` | Runs the application |
| Container | `employee-laptop` | Container inside the Pod |
| Image | `nginx` | Docker image used to create the container |
| Label | `department=web-team` | Groups related Pods |
| Selector | `department=web-team` | Finds Pods with matching labels |

### Key Takeaway

- **Names** uniquely identify Kubernetes objects.
- **Labels** group related objects together.
- **Selectors** use labels to find and manage the correct Pods.
- **Images** define what software runs inside the container.
- **Containers** run inside Pods.
- **Deployments** create ReplicaSets.
- **ReplicaSets** create and maintain Pods.

---

## 3. Benefits of ReplicaSets

1. Deleting a pod causes it to be re-created automatically.
2. Good for scaling.
3. Maintains a stable set of replica pods at any time.
4. RS are primarily designed to maintain a specific state of running pods — not to manage regular updates or changes to their config.

---

## 4. Scaling a ReplicaSet

### Step 1: Create a ReplicaSet Manifest (`replicaset.yaml`)

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: frontend-replicaset
spec:
  replicas: 3
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
      - name: php-redis
        image: us-docker.pkg.dev/google-samples/containers/gke/gb-frontend:v5
```

```bash
kubectl apply -f replicaset.yaml
```

### Step 2: Verify Creation

```bash
kubectl get replicaset
kubectl get pods
kubectl get pods --show-labels
# NAME                        READY   STATUS    RESTARTS   AGE   LABELS
# frontend-replicaset-2957q   1/1     Running   0          60s   tier=frontend
# frontend-replicaset-4jbpr   1/1     Running   0          60s   tier=frontend
# frontend-replicaset-9v88g   1/1     Running   0          60s   tier=frontend
```

### Step 3: Scale Up

```bash
k scale --replicas=5 rs frontend-replicaset
# replicaset.apps/frontend-replicaset scaled

k get rs
# NAME                  DESIRED   CURRENT   READY   AGE
# frontend-replicaset   5         5         5       2m57s

k get pods
# NAME                        READY   STATUS    RESTARTS   AGE
# frontend-replicaset-2957q   1/1     Running   0          2m59s
# frontend-replicaset-4jbpr   1/1     Running   0          2m59s
# frontend-replicaset-9v88g   1/1     Running   0          2m59s
# frontend-replicaset-jvt75   1/1     Running   0          75s
# frontend-replicaset-vt5b5   1/1     Running   0          75s
```

### Scale Down

```bash
k scale --replicas=1 rs frontend-replicaset
# replicaset.apps/frontend-replicaset scaled

k get rs
# NAME                  DESIRED   CURRENT   READY   AGE
# frontend-replicaset   1         1         1       3m25s

k get pods
# NAME                        READY   STATUS    RESTARTS   AGE
# frontend-replicaset-2957q   1/1     Running   0          3m27s
```

### Step 4: Delete the ReplicaSet

```bash
kubectl delete -f replicaset.yaml
```

---

## 5. Known Challenges with ReplicaSets

### Challenge 1 — Updating Container Image Doesn't Update Existing Pods

When you update the pod template (e.g. change the container image) in an RS, the **existing pods are NOT updated**.

```bash
vi rs.yaml
kubectl apply -f rs.yaml
kubectl get rs
kubectl get pods
```

Now change the image from `nginx` to `httpd`:

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: webserver-replicaset
spec:
  replicas: 3
  selector:
    matchLabels:
      app: webserver
  template:
    metadata:
      labels:
        app: webserver
    spec:
      containers:
      - name: nginx-container
        image: httpd:latest
```

```bash
k apply -f rs.yaml
k get pods   # shows old pod. Image not updated
kubectl describe rs webserver-replicaset
```

```bash
k describe pod webserver-replicaset-fkcxp | grep -i image:
# Image:          nginx:latest
```

The image is still the old one. To force pods to pick up the new image, scale down to 0 and back up:

```bash
kubectl scale rs/webserver-replicaset --replicas=0
kubectl scale rs/webserver-replicaset --replicas=3
```

Now the new image is used:

```bash
k describe pod webserver-replicaset-vgzrp | grep -i image:
# Image:          httpd:latest
```

### Challenge 2 — No Built-in Rollback Mechanism

ReplicaSets lack a rollback mechanism for reverting to a previous config in case of an error during an update.

### Challenge 3 — Label Collision with RS Selectors

When an RS selector matches labels of pods it didn't create, it starts treating those pods as part of its managed set. This can cause unintended consequences.

```bash
kubectl delete -f rs.yaml
```

Create a standalone pod and give it a matching label:

```bash
k run external-pod --image=nginx
# pod/external-pod created

k get pod
# NAME           READY   STATUS    RESTARTS   AGE
# external-pod   1/1     Running   0          4s

k label pod external-pod app=webserver
# pod/external-pod labeled

k get pods --show-labels
# NAME           READY   STATUS    RESTARTS   AGE   LABELS
# external-pod   1/1     Running   0          36s   app=webserver,run=external-pod
```

Apply the ReplicaSet whose selector matches `app=webserver`:

```bash
k apply -f rs.yaml
# replicaset.apps/webserver-replicaset created
```

Because of the label collision, only 2 new pods are created (the RS counts `external-pod` toward its desired count of 3):

```bash
k get pods
# NAME                         READY   STATUS    RESTARTS   AGE
# external-pod                 1/1     Running   0          60s
# webserver-replicaset-glrcb   1/1     Running   0          18s
# webserver-replicaset-zkm6h   1/1     Running   0          18s
```

```bash
kubectl delete -f rs.yaml   # will delete all 3 pods, including external-pod
kubectl get pods
```

---

## Quick Reference

| Command | Purpose |
|---|---|
| `k apply -f replica.yaml` | Create/update the ReplicaSet from a manifest |
| `k get pods` | List pods managed by the ReplicaSet |
| `k get rs` | List ReplicaSets (desired/current/ready counts) |
| `k describe rs <name>` | Detailed ReplicaSet info + events |
| `k get pods <pod> -o yaml` | Full YAML for a single pod |
| `k get pods --show-labels` | List pods with their labels |
| `k get pods -l <key>=<value>` | Filter pods by label |
| `kubectl describe pod <pod> \| grep -i labe` | Show labels for a single pod |
| `k scale --replicas=<n> rs <name>` | Scale a ReplicaSet up or down |
| `k label pod <pod> <key>=<value>` | Add/update a label on a pod |
| `k delete rs <name>` | Delete the ReplicaSet (and its pods) |
| `kubectl delete -f <file>.yaml` | Delete resources defined in a manifest |
