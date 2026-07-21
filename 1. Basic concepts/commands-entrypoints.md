# Commands and Arguments in Kubernetes

**Documentation Referenced:** [https://hub.docker.com/_/nginx](https://hub.docker.com/_/nginx)

---

## Docker Background: CMD vs ENTRYPOINT

### Goal

Create an image that just pings google.com 3 times. When the pod is spun up, it performs the action and exits.

### Dockerfile — Example 1 (CMD only)

```bash
vi Dockerfile
```

```dockerfile
FROM busybox:latest
CMD ["ping","-c","3","google.com"]
```

Build and run:

```bash
docker build -t busybox:ping .
docker images
```

```
PS D:\K8s> docker run busybox:ping
PING google.com (142.250.151.113): 56 data bytes
64 bytes from 142.250.151.113: seq=0 ttl=63 time=74.687 ms
64 bytes from 142.250.151.113: seq=1 ttl=63 time=13.188 ms
64 bytes from 142.250.151.113: seq=2 ttl=63 time=16.995 ms
```

```bash
docker ps
```
> Doesn't show the container — it's already exited.

```bash
docker ps -a
```
> Shows the exited container.

### Dockerfile — Example 2 (ENTRYPOINT + CMD)

```dockerfile
FROM ubuntu
ENTRYPOINT ["/bin/echo"]
CMD ["hello", "world"]
```

Build and run:

```bash
docker build -t ubuntu:custom .
docker images
docker run ubuntu:custom
```

### Overriding the Default CMD in Docker

```bash
docker run ubuntu:custom how are you
```

> `ENTRYPOINT` stays fixed (`/bin/echo`), but the `CMD` args (`hello world`) get replaced by whatever you pass on the command line (`how are you`).

---

## Kubernetes: Commands and Arguments

### Example 1 — Echo Pod

```bash
kubectl run echo-pod --image=busybox:latest --command -- "Hello World!"
kubectl logs echo-pod
kubectl delete pod echo-pod
```

### Example 2 — Ping Pod

```bash
kubectl run ping-pod --image=busybox:latest --command -- ping "-c" "30" google.com
kubectl logs ping-pod
kubectl delete pod ping-pod
```

---

### Example 3 — Manifest File with `command` and `args`

**Base Pod Manifest (reference)**

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

**Final Pod Manifest with CMD and Args** — `cmd-args.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: new-ping-pod
spec:
  containers:
  - name: ping-container
    image: busybox:latest
    command: ["ping"]
    args: ["-c","60","google.com"]
```

Apply and check:

```bash
kubectl apply -f cmd-args.yaml
kubectl get pods
kubectl logs new-ping-pod
```

---

### Example 4 — Using Only `command` (no `args`)

`cmd-clarity.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: new-ping-pod
spec:
  containers:
  - name: ping-container
    image: busybox:latest
    command: ["ping","-c","60","google.com"]
    args: []
```

```bash
kubectl apply -f cmd-clarity.yaml
kubectl get pods
kubectl logs new-ping-pod
kubectl delete pod new-ping-pod
```

---

### Example 5 — Specifying `command` in List (multi-line) Format

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: new-ping-pod
spec:
  containers:
  - name: ping-container
    image: busybox:latest
    command:
      - "ping"
      - "-c"
      - "60"
      - "google.com"
```

```bash
kubectl apply -f cmd-clarity.yaml
kubectl get pods
kubectl logs new-ping-pod
kubectl delete pod new-ping-pod
```

---

## Key Concepts: Docker vs Kubernetes Mapping

| Docker | Kubernetes | Purpose |
|---|---|---|
| `ENTRYPOINT` | `command` | The fixed executable that runs in the container |
| `CMD` | `args` | Default arguments; can be overridden |

- If a Kubernetes pod spec sets `command`, it **overrides** the image's `ENTRYPOINT`.
- If a Kubernetes pod spec sets `args`, it **overrides** the image's `CMD`.
- `command` can be written as a single-line array (`["ping","-c","60","google.com"]`) or as a multi-line YAML list — both are equivalent.

---

## Quick Reference

| Command | Purpose |
|---|---|
| `docker build -t <name>:<tag> .` | Build an image from a Dockerfile |
| `docker run <image>` | Run a container from an image |
| `docker ps` / `docker ps -a` | List running / all containers |
| `kubectl run <pod> --image=<image> --command -- <cmd>` | Imperatively create a pod with a custom command |
| `kubectl apply -f <file.yaml>` | Create/update a pod from a manifest |
| `kubectl logs <pod>` | View pod output/logs |
| `kubectl delete pod <pod>` | Delete a pod |
