# Multi Container Pod

## Pod Manifest

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multi-container-pod
spec:
  containers:
  - name: nginx-container
    image: nginx
  - name: redis-container
    image: redis
```

Create the pod:

```bash
k apply -f multi.yaml
# pod/multi-container-pod created
```

Check pod status:

```bash
k get pods
# NAME                  READY   STATUS              RESTARTS   AGE
# multi-container-pod   0/2     ContainerCreating   0          3s
```

Note the `0/2` — both containers need to be ready before the pod shows fully ready.

## Describe the Pod

```bash
k describe pod multi-container-pod
```

Key details returned:

- **Status:** Running
- **Node:** node01
- **Containers:** nginx-container (image: `nginx`) and redis-container (image: `redis`), each with its own container ID, state, and ready status
- **QoS Class:** BestEffort
- **Volumes:** a single shared projected `kube-api-access` volume mounted into both containers

Events show each container's image pull → create → start sequence happening independently — nginx-container starts first, then redis-container.

## Exec Into a Multi-Container Pod

With more than one container, you need to specify **which** container to shell into.

Without `-c`, it defaults to the first container defined in the spec:

```bash
k exec -it multi-container-pod -- /bin/bash
# Defaulted container "nginx-container" out of: nginx-container, redis-container
```

To target a specific container, use `-c`:

```bash
kubectl exec -it multi-container-pod -c redis-container -- bash
```

## List All Containers in a Pod

Using `describe` + `grep`:

```bash
k describe pod multi-container-pod | grep -i container:
# nginx-container:
# redis-container:
```

Using `jsonpath` (cleaner for scripting):

```bash
kubectl get pod multi-container-pod -o jsonpath='{.spec.containers[*].name}'
# nginx-container redis-container
```

## Delete the Pod

```bash
k delete pod multi-container-pod
# pod "multi-container-pod" deleted from default namespace
```

---

## Quick Reference

| Command | Purpose |
|---|---|
| `k apply -f multi.yaml` | Create/update the pod from a manifest |
| `k get pods` | List pods and their status |
| `k describe pod <pod>` | Detailed pod info + events for all containers |
| `k exec -it <pod> -- /bin/bash` | Shell into the default (first) container |
| `k exec -it <pod> -c <container> -- bash` | Shell into a specific container |
| `k describe pod <pod> \| grep -i container:` | List container names |
| `k get pod <pod> -o jsonpath='{.spec.containers[*].name}'` | List container names (jsonpath) |
| `k delete pod <pod>` | Delete the pod |
