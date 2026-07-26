# Kubernetes Deployment

Official Documentation:
https://kubernetes.io/docs/concepts/workloads/controllers/deployment/

---

# What is a Deployment?

A **Deployment** is a Kubernetes object used to manage Pods in a declarative way.

A Deployment creates and manages a **ReplicaSet**, and the ReplicaSet creates and manages the Pods.

```
Deployment
      │
      ▼
 ReplicaSet
      │
      ▼
    Pods
```

> **ReplicaSet and Deployment always go hand in hand.**

A Deployment is a **higher-level abstraction built on top of a ReplicaSet**.

Besides maintaining the desired number of Pods, it also provides:

- Rolling Updates
- Rollbacks
- Version History
- Scaling
- Self-Healing (through ReplicaSet)

---

# Real World Analogy

Imagine an IT company.

```
CEO
 │
 ▼
HR Manager (Deployment)
 │
 ▼
Team Leader (ReplicaSet)
 │
 ▼
Employees (Pods)
```

- Deployment decides what version should run.
- ReplicaSet ensures the required number of Pods exist.
- Pods are the actual running containers.

---

# Why not use ReplicaSet directly?

ReplicaSet only guarantees the required number of Pods are running.

If you change the container image inside a ReplicaSet:

- Existing Pods are **NOT** updated.
- Only newly created Pods use the new image.

Deployment solves this problem by performing a **Rolling Update**.

---

# Deployment YAML Example

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: nginx-deployment
  labels:
    app: nginx

spec:
  replicas: 3

  selector:
    matchLabels:
      app: nginx

  template:
    metadata:
      labels:
        app: nginx

    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

Apply it

```bash
kubectl apply -f deployment.yaml
```

---

# Generate Deployment YAML

```bash
kubectl create deployment nginx-deployment \
--image=nginx \
--dry-run=client \
-o yaml > deployment.yaml
```

---

# Verify Deployment

```bash
kubectl get deployment
```

Example

```text
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3/3     3            3           23s
```

---

Check Pods

```bash
kubectl get pods
```

```text
NAME                                READY   STATUS
nginx-deployment-77bc6bd484-85v7d   1/1     Running
nginx-deployment-77bc6bd484-pvsbk   1/1     Running
nginx-deployment-77bc6bd484-t82fk   1/1     Running
```

---

Check ReplicaSets

```bash
kubectl get rs
```

```text
NAME                          DESIRED   CURRENT   READY
nginx-deployment-77bc6bd484   3         3         3
```

Notice:

```
Deployment
      │
      ▼
ReplicaSet
      │
      ▼
3 Pods
```

---

# Case Study - Updating the Container Image

Suppose the Deployment currently uses

```yaml
image: nginx:1.14.2
```

Now change it to

```yaml
image: httpd:latest
```

Apply the changes.

```bash
kubectl apply -f deployment.yaml
```

---

## What happens internally?

Deployment creates a **new ReplicaSet**.

Old ReplicaSet:

```
nginx:1.14.2
```

New ReplicaSet:

```
httpd:latest
```

Then Kubernetes performs a **Rolling Update**.

```
Old ReplicaSet
3 Pods
   │
   ▼
2 Pods
   │
   ▼
1 Pod
   │
   ▼
0 Pods

At the same time

New ReplicaSet

0 Pods
   │
   ▼
1 Pod
   │
   ▼
2 Pods
   │
   ▼
3 Pods
```

No downtime.

---

## Verify Deployment

```bash
kubectl get deployment
```

During update

```text
NAME               READY   UP-TO-DATE   AVAILABLE
nginx-deployment   3/3     2            3
```

---

## Check Pods

```bash
kubectl get pods
```

Initially

```text
nginx-deployment-77bc6bd484-pvsbk
nginx-deployment-869f69485c-gxpcd
nginx-deployment-869f69485c-qc8kd
nginx-deployment-869f69485c-tjjcz
```

Notice Pods from **both ReplicaSets** exist temporarily.

Once the rollout finishes

```text
nginx-deployment-869f69485c-gxpcd
nginx-deployment-869f69485c-qc8kd
nginx-deployment-869f69485c-tjjcz
```

Old Pods disappear.

---

## Check ReplicaSets

```bash
kubectl get rs
```

```text
NAME                          DESIRED
nginx-deployment-77bc6bd484   0
nginx-deployment-869f69485c   3
```

Deployment keeps the old ReplicaSet for rollback.

---

## Verify Image

```bash
kubectl describe deployment nginx-deployment | grep -i image
```

Output

```text
Image: httpd:latest
```

---

# Rollback

One of the biggest advantages of Deployments is rollback.

View rollout history.

```bash
kubectl rollout history deployment/nginx-deployment
```

Output

```text
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
```

---

Rollback to the previous version.

```bash
kubectl rollout undo deployment nginx-deployment
```

Output

```text
deployment.apps/nginx-deployment rolled back
```

---

Check ReplicaSets

```bash
kubectl get rs
```

```text
NAME                          DESIRED
nginx-deployment-77bc6bd484   3
nginx-deployment-869f69485c   0
```

Deployment simply switches back to the previous ReplicaSet.

---

Verify Image

```bash
kubectl describe deployment nginx-deployment | grep -i image
```

Output

```text
Image: nginx:1.14.2
```

---

# Rollback to a Specific Revision

Rollback again to Revision 2.

```bash
kubectl rollout undo deployment nginx-deployment --to-revision=2
```

Verify

```bash
kubectl describe deployment nginx-deployment | grep -i image
```

Output

```text
Image: httpd:latest
```

---

# Scaling a Deployment

Scale to 3 Pods.

```bash
kubectl scale --replicas=3 deployment nginx-deployment
```

Scale to 5 Pods.

```bash
kubectl scale --replicas=5 deployment nginx-deployment
```

ReplicaSet automatically creates or removes Pods.

---

# Delete Deployment

```bash
kubectl delete deployment nginx-deployment
```

Verify

```bash
kubectl get deployment
kubectl get rs
kubectl get pods
```

Everything created by the Deployment is removed.

---

# ReplicaSet vs Deployment

| Feature | ReplicaSet | Deployment |
|----------|------------|------------|
| Abstraction Level | Lower | Higher |
| Manages Pods | ✅ | ✅ (through ReplicaSet) |
| Rolling Updates | ❌ | ✅ |
| Rollbacks | ❌ | ✅ |
| Version History | ❌ | ✅ |
| Scaling | Manual | Easy |
| Recommended for Production | ❌ | ✅ |

---

# Common kubectl Commands

## Create Deployment YAML

```bash
kubectl create deployment my-deployment \
--image=nginx \
--dry-run=client \
-o yaml
```

---

## Create Deployment YAML with 3 Replicas

```bash
kubectl create deployment my-deployment \
--image=nginx \
--replicas=3 \
--dry-run=client \
-o yaml
```

---

## Create Deployment

```bash
kubectl create deployment my-deployment \
--image=nginx \
--replicas=3
```

---

## View Deployments

```bash
kubectl get deployment
```

---

## View Pods

```bash
kubectl get pods
```

---

## View ReplicaSets

```bash
kubectl get rs
```

---

## Update Deployment

```bash
kubectl apply -f deployment.yaml
```

---

## Check Rollout Status

```bash
kubectl rollout status deployment/nginx-deployment
```

---

## Rollout History

```bash
kubectl rollout history deployment/nginx-deployment
```

---

## Rollback

```bash
kubectl rollout undo deployment/nginx-deployment
```

---

## Rollback to Specific Revision

```bash
kubectl rollout undo deployment/nginx-deployment --to-revision=2
```

---

## Scale Deployment

```bash
kubectl scale deployment nginx-deployment --replicas=5
```

---

## Delete Deployment

```bash
kubectl delete deployment nginx-deployment
```

---

# Key Takeaways

- Deployment is built on top of a ReplicaSet.
- ReplicaSet manages Pods.
- Deployment manages ReplicaSets.
- Deployments support rolling updates with zero/minimal downtime.
- Deployments maintain version history.
- Rollbacks are simple using `kubectl rollout undo`.
- Scaling is easy using `kubectl scale`.
- In production, **always use Deployments instead of creating ReplicaSets directly**.
