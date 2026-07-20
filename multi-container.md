# Kubernetes Pod Manifests

## Base Pod (Single Container)

`pod.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx-container
    image: nginx
```

Apply the manifest:

```bash
kubectl apply -f pod.yaml
```

---

## Multi-Container Pod

`multi-container-pod.yaml`

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

Apply the manifest:

```bash
kubectl apply -f multi-container-pod.yaml
```

### Verify the Pod

```bash
kubectl get pods
```

```
PS D:\K8s> kubectl get pods
NAME                  READY   STATUS    RESTARTS   AGE
multi-container-pod   2/2     Running   0          51s
PS D:\K8s>
```

> Note `READY 2/2` — both containers in the pod are running.

### Describe the Pod

```bash
kubectl describe pod multi-container-pod
```

---

## Exec into a Multi-Container Pod

```bash
kubectl exec -it multi-container-pod -- bash
```

```
PS D:\K8s> kubectl get pods
NAME                  READY   STATUS    RESTARTS   AGE
multi-container-pod   2/2     Running   0          3m18s
PS D:\K8s> kubectl exec -it multi-container-pod -- bash
Defaulted container "nginx-container" out of: nginx-container, redis-container
root@multi-container-pod:/#
```

> By default, `kubectl exec` connects to the **first container** defined in the pod spec (`nginx-container` in this case).

### Exec into a Specific Container

You can explicitly specify which container to log into using `-c`:

```bash
kubectl exec -it multi-container-pod -c redis-container -- bash
```

---

## Delete the Resources

```bash
kubectl delete -f pod.yaml
kubectl delete -f multi-container-pod.yaml
```

---

## Quick Reference

| Command | Purpose |
|---|---|
| `kubectl apply -f <file.yaml>` | Create/update resource from manifest |
| `kubectl get pods` | List pods and their status |
| `kubectl describe pod <name>` | Detailed pod info (containers, events, etc.) |
| `kubectl exec -it <pod> -- bash` | Shell into default (first) container |
| `kubectl exec -it <pod> -c <container> -- bash` | Shell into a specific container |
| `kubectl delete -f <file.yaml>` | Delete resources defined in a manifest |
