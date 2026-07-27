# ReplicaSet

Docs: https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/

## Manifest

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

## Check Pods

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

## Check the ReplicaSet

```bash
k get rs
# NAME       DESIRED   CURRENT   READY   AGE
# frontend   3         3         3       63s
```

## Describe the ReplicaSet

```bash
k describe rs frontend
```

Key details returned:

- **Selector:** `tier=frontend` — how the ReplicaSet knows which pods it owns
- **Replicas:** 3 current / 3 desired
- **Pods Status:** 3 Running / 0 Waiting / 0 Succeeded / 0 Failed
- **Pod Template:** the container spec (`php-redis` image) used to create each pod

Events show a `SuccessfulCreate` entry for each of the three pods.

## Info on One Pod

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

## Delete the ReplicaSet

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

## Quick Reference

| Command | Purpose |
|---|---|
| `k apply -f replica.yaml` | Create/update the ReplicaSet from a manifest |
| `k get pods` | List pods managed by the ReplicaSet |
| `k get rs` | List ReplicaSets (desired/current/ready counts) |
| `k describe rs <name>` | Detailed ReplicaSet info + events |
| `k get pods <pod> -o yaml` | Full YAML for a single pod |
| `k get pods --show-labels` | List pods with their labels |
| `k describe pod <pod> \| grep -i labe` | Show labels for a single pod |
| `k delete rs <name>` | Delete the ReplicaSet (and its pods) |
