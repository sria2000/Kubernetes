# Pod Adoption by ReplicaSet (Non-Template Pod Acquisition)

> **Reference:** https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/

## Objective

Understand how a **ReplicaSet can adopt existing Pods** if their labels match the ReplicaSet's selector.

A ReplicaSet does **not** create new Pods if the desired number of matching Pods already exists, even if those Pods were created manually.

---

## Step 1: Create Two Standalone Pods

Create a file named `pod.yaml`.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod1
  labels:
    tier: frontend
spec:
  containers:
  - name: hello1
    image: gcr.io/google-samples/hello-app:2.0

---
apiVersion: v1
kind: Pod
metadata:
  name: pod2
  labels:
    tier: frontend
spec:
  containers:
  - name: hello2
    image: gcr.io/google-samples/hello-app:1.0
```

Apply the manifest:

```bash
kubectl apply -f pod.yaml
```

Verify:

```bash
kubectl get pods
```

Example output:

```text
NAME   READY   STATUS    RESTARTS   AGE
pod1   1/1     Running   0          2m
pod2   1/1     Running   0          2m
```

---

## Step 2: Create a ReplicaSet

Create a file named `replicaset.yaml`.

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: frontend
  labels:
    app: guestbook
    tier: frontend

spec:
  replicas: 2

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

Apply the ReplicaSet:

```bash
kubectl apply -f replicaset.yaml
```

Check the ReplicaSet:

```bash
kubectl get rs
```

Example output:

```text
NAME       DESIRED   CURRENT   READY   AGE
frontend   2         2         2       1m
```

---

## Step 3: Verify the ReplicaSet Did NOT Create New Pods

Check the Pods:

```bash
kubectl get pods
```

Output:

```text
NAME   READY   STATUS    RESTARTS   AGE
pod1   1/1     Running   0          3m
pod2   1/1     Running   0          3m
```

Notice that there are still only **2 Pods**.

The ReplicaSet **did not create** Pods from its template because:

- Desired replicas = **2**
- Existing Pods matching `tier=frontend` = **2**

The ReplicaSet **adopted** the existing Pods.

---

## Step 4: Delete the Existing Pods

Delete the Pods one by one:

```bash
kubectl delete pod pod1
kubectl delete pod pod2
```

Immediately check again:

```bash
kubectl get pods
```

Output:

```text
NAME             READY   STATUS    RESTARTS   AGE
frontend-2mzll   1/1     Running   0          3s
frontend-wg4kf   1/1     Running   0          5s
```

These new Pods were created from the ReplicaSet template.

---

# How ReplicaSet Decides

```text
ReplicaSet starts
        │
        ▼
Looks for Pods matching selector
        │
        ▼
tier=frontend
        │
        ├── Finds 2 Pods
        │        │
        │        └── Desired replicas already met
        │
        └── No new Pods created
```

After deleting the Pods:

```text
ReplicaSet
        │
        ▼
Matching Pods = 0
        │
        ▼
Needs 2 Pods
        │
        ▼
Creates 2 new Pods from the template
```

---

# Why This Happens

The ReplicaSet selector is:

```yaml
selector:
  matchLabels:
    tier: frontend
```

The manually created Pods also have:

```yaml
labels:
  tier: frontend
```

Because the labels match, the ReplicaSet considers those Pods as part of its desired state and adopts them instead of creating new ones.

---

# Verify ReplicaSet Ownership

You can verify that the ReplicaSet has adopted the Pods:

```bash
kubectl describe rs frontend
```

or inspect a Pod:

```bash
kubectl describe pod pod1
```

Look for the **Controlled By** field:

```text
Controlled By:  ReplicaSet/frontend
```

---

# Key Points

- ReplicaSets manage Pods based on **label selectors**.
- Existing Pods with matching labels are **adopted**.
- ReplicaSets create new Pods **only if the number of matching Pods is less than the desired replica count**.
- The Pod template is used **only when new Pods need to be created**.

---

# CKA Exam Tip

Remember this rule:

> **ReplicaSets care about labels, not how the Pods were created.**

If a manually created Pod matches the ReplicaSet selector, the ReplicaSet can adopt it instead of creating a new Pod.
