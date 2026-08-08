# Kubernetes Labels & Selectors

## Overview

Labels and selectors are one of the most important concepts in Kubernetes.

They are used to **organize, group, and filter Kubernetes objects**.

---

# Labels

A **Label** is a key-value pair attached to Kubernetes objects.

Example:

```yaml
env: dev
```

Labels help identify and group resources such as:

- Pods
- Services
- Deployments
- ReplicaSets
- DaemonSets
- Secrets
- Namespaces

---

## Real World Example (AWS)

In AWS, if you want to identify DEV EC2 instances:

Example EC2 Tag:

```
Environment = DEV
```

You can filter EC2 instances using this tag.

Kubernetes works in a similar way:

```
AWS Tag
   |
   |
Kubernetes Label
```

Example:

```yaml
env: dev
```

A Pod with this label belongs to the DEV environment.

---

# Selectors

Selectors allow Kubernetes users to **filter objects based on labels**.

Example:

Show all Pods with:

```
env=dev
```

Command:

```bash
kubectl get pods -l env=dev
```

Output:

```text
NAME
pod-1
```

---

# Kubernetes Objects That Support Labels

Labels can be attached to many Kubernetes resources:

1. Pods
2. Services
3. Secrets
4. Namespaces
5. Deployments
6. DaemonSets
7. ReplicaSets

---

# Hands-on Lab

## Step 1: Create 3 Pods

Create three nginx Pods:

```bash
kubectl run pod-1 --image=nginx

kubectl run pod-2 --image=nginx

kubectl run pod-3 --image=nginx
```

Check:

```bash
kubectl get pods
```

Output:

```text
NAME    READY   STATUS
pod-1   1/1     Running
pod-2   1/1     Running
pod-3   1/1     Running
```

---

# Step 2: Display Existing Labels

```bash
kubectl get pods --show-labels
```

Output:

```text
NAME    READY   STATUS    LABELS
pod-1   1/1     Running   run=pod-1
pod-2   1/1     Running   run=pod-2
pod-3   1/1     Running   run=pod-3
```

Kubernetes automatically added:

```
run=<pod-name>
```

---

# Step 3: Add Environment Labels

Assign environment labels:

```bash
kubectl label pod pod-1 env=dev

kubectl label pod pod-2 env=uat

kubectl label pod pod-3 env=prod
```

Verify:

```bash
kubectl get pods --show-labels
```

Output:

```text
NAME    LABELS
pod-1   env=dev,run=pod-1
pod-2   env=uat,run=pod-2
pod-3   env=prod,run=pod-3
```

---

# Step 4: Use Selectors to Filter Pods

## Show DEV Pods

```bash
kubectl get pods -l env=dev
```

Output:

```text
NAME
pod-1
```

---

## Show UAT Pods

```bash
kubectl get pods -l env=uat
```

Output:

```text
NAME
pod-2
```

---

## Show Everything Except DEV

```bash
kubectl get pods -l env!=dev
```

Output:

```text
NAME
pod-2
pod-3
```

---

# Step 5: Check Label Command Help

```bash
kubectl label --help
```

Useful options:

```text
kubectl label <resource> <name> key=value
```

---

# Step 6: Remove a Label

Current labels:

```bash
kubectl get pods --show-labels
```

Example:

```text
pod-1 env=dev
```

Remove the label:

```bash
kubectl label pod pod-1 env-
```

Output:

```text
pod/pod-1 unlabeled
```

Verify:

```bash
kubectl get pods --show-labels
```

Output:

```text
pod-1 run=pod-1
```

The `env` label has been removed.

---

# Step 7: Generate Pod YAML

Generate a Pod manifest:

```bash
kubectl run nginx \
--image=nginx \
--dry-run=client \
-o yaml
```

Output:

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    name: nginx
```

Notice:

Labels are stored under:

```yaml
metadata:
  labels:
```

---

# Step 8: Add Labels Using YAML

Generate YAML:

```bash
kubectl run nginx \
--image=nginx \
--dry-run=client \
-o yaml > label-pod.yaml
```

Edit the file:

```yaml
metadata:
  labels:
    run: nginx
    env: dev
```

Final:

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: nginx
    env: dev
  name: nginx

spec:
  containers:
  - image: nginx
    name: nginx
```

Apply:

```bash
kubectl apply -f label-pod.yaml
```

Check:

```bash
kubectl get pods --show-labels
```

Output:

```text
NAME
nginx

LABELS
env=dev,run=nginx
```

---

# Step 9: Add Label to All Pods

Add a label to every Pod:

```bash
kubectl label pods --all status=dev
```

Verify:

```bash
kubectl get pods --show-labels
```

Example:

```text
nginx   env=dev,run=nginx,status=dev

pod-1   run=pod-1,status=dev

pod-2   env=uat,run=pod-2,status=dev

pod-3   env=prod,run=pod-3,status=dev
```

---

# Remove Label from All Pods

Remove the `status` label:

```bash
kubectl label pods --all status-
```

Verify:

```bash
kubectl get pods --show-labels
```

Output:

```text
nginx   env=dev,run=nginx

pod-1   run=pod-1

pod-2   env=uat,run=pod-2

pod-3   env=prod,run=pod-3
```

---

# Step 10: Delete All Pods

Cleanup:

```bash
kubectl delete pods --all
```

---

# Label Selector Examples

## Equality Based Selector

Find DEV Pods:

```bash
kubectl get pods -l env=dev
```

---

## Not Equal Selector

Find non-DEV Pods:

```bash
kubectl get pods -l env!=dev
```

---

## Multiple Labels

Example:

```bash
kubectl get pods -l env=dev,status=active
```

Both labels must match.

---

# Labels vs Selectors

| Feature | Description |
|---|---|
| Label | Key-value pair attached to an object |
| Selector | Used to search/filter objects using labels |
| Example Label | `env=dev` |
| Example Selector | `-l env=dev` |

---

# CKA Exam Tips

Remember:

- Labels identify Kubernetes objects.
- Selectors find objects using labels.
- Services use selectors to find Pods.
- ReplicaSets use selectors to manage Pods.
- Deployments use selectors to manage ReplicaSets.

Important relationship:

```
Deployment
     |
     ▼
ReplicaSet
     |
     ▼
Pods

Selector
     |
     ▼
Labels
```

---

# Key Takeaway

> Kubernetes does not understand application names. It uses labels and selectors to group and manage resources.
