# Single Container Pod

## link from kubernets.io -- https://kubernetes.io/docs/concepts/workloads/pods/

## Pod Manifest

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

Create the pod:

```bash
k apply -f single.yaml
# pod/nginx created
```

Check pod status:

```bash
k get pods
# NAME    READY   STATUS              RESTARTS   AGE
# nginx   0/1     ContainerCreating   0          4s
```

## Check Logs

```bash
k logs nginx
```

Shows the nginx entrypoint scripts running, IPv6 listen config being applied, worker process tuning, and finally nginx starting up and listening for connections.

## Describe the Pod

```bash
k describe pod nginx
```

Key details returned:

- **Status:** Running
- **Node:** node01
- **Container:** nginx-container (image: `nginx`)
- **Ready:** True
- **Restart Count:** 0
- **QoS Class:** BestEffort
- **Volumes:** a projected `kube-api-access` volume mounting the service account token, CA cert, and downward API data

Events show the scheduling → image pull → container creation → container start sequence, useful for troubleshooting slow or failed pod starts.

## Exec Into the Pod

```bash
k exec -it nginx -- /bin/bash
```

Drops into a shell inside the container — useful for verifying the running OS/kernel, checking installed tools, or debugging from inside.

```bash
uname -a
```

## Delete the Pod

```bash
k delete pod nginx
# pod "nginx" deleted from default namespace
```

Confirm it's gone:

```bash
k get pods
# No resources found in default namespace.
```

---

## Quick Reference

| Command | Purpose |
|---|---|
| `k apply -f single.yaml` | Create/update the pod from a manifest |
| `k get pods` | List pods and their status |
| `k logs <pod>` | View container logs |
| `k describe pod <pod>` | Detailed pod info + events |
| `k exec -it <pod> -- /bin/bash` | Shell into the running container |
| `k delete pod <pod>` | Delete the pod |
