# 13. A Pod is not running. What are the steps to debug the issue?

Demo scenario: forced a real failure with a bad image tag (`nginx:brokenpod`).

```yaml
# sripod.yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: sripod
  name: sripod
spec:
  containers:
  - image: nginx:brokenpod
    name: sripod
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

```
root@controlplane:~$ k apply -f sripod.yaml
pod/sripod created
root@controlplane:~$ k get pod
NAME     READY   STATUS              RESTARTS   AGE
sripod   0/1     ContainerCreating   0          3s
root@controlplane:~$
```

### a) Check pod status — shows ImagePullBackOff
```
root@controlplane:~$ k get pod
NAME     READY   STATUS             RESTARTS   AGE
sripod   0/1     ImagePullBackOff   0          26s
root@controlplane:~$
```

### b) Describe the pod — Events section reveals the root cause
```
root@controlplane:~$ k describe pod sripod
Name:             sripod
Namespace:        default
Priority:         0
Service Account:  default
Node:             node01/172.30.2.2
Start Time:       Mon, 27 Jul 2026 17:18:13 +0000
Labels:           run=sripod
Annotations:      <none>
Status:           Pending
IP:               192.168.1.121
IPs:
  IP:  192.168.1.121
Containers:
  sripod:
    Container ID:
    Image:          nginx:brokenpod
    Image ID:
    Port:           <none>
    Host Port:      <none>
    State:          Waiting
      Reason:       ImagePullBackOff
    Ready:          False
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-lkhpz (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True
  Initialized                 True
  Ready                       False
  ContainersReady             False
  PodScheduled                True
Volumes:
  kube-api-access-lkhpz:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    Optional:                false
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type     Reason     Age                From               Message
  ----     ------     ----               ----               -------
  Normal   Scheduled  77s                default-scheduler  Successfully assigned default/sripod to node01
  Normal   Pulling    35s (x3 over 76s)  kubelet            spec.containers{sripod}: Pulling image "nginx:brokenpod"
  Warning  Failed     34s (x3 over 74s)  kubelet            spec.containers{sripod}: Failed to pull image "nginx:brokenpod": rpc error: code = NotFound desc = failed to pull and unpack image "docker.io/library/nginx:brokenpod": failed to resolve image: docker.io/library/nginx:brokenpod: not found
  Warning  Failed     34s (x3 over 74s)  kubelet            spec.containers{sripod}: Error: ErrImagePull
  Normal   BackOff    7s (x4 over 74s)   kubelet            spec.containers{sripod}: Back-off pulling image "nginx:brokenpod"
  Warning  Failed     7s (x4 over 74s)   kubelet            spec.containers{sripod}: Error: ImagePullBackOff
root@controlplane:~$
```
**PS:** the describe output clearly shows the problem is with the image.

### c) Check logs
```
root@controlplane:~$ k logs sripod
Error from server (BadRequest): container "sripod" in pod "sripod" is waiting to start: trying and failing to pull image
root@controlplane:~$
```
`kubectl logs` is not useful here — the container never started, so there's nothing to log.

### d) Narrow down the problem
```
root@controlplane:~$ k describe pod sripod | grep -i ngin
    Image:          nginx:brokenpod
  Normal   Pulling    73s (x4 over 2m36s)  kubelet            spec.containers{sripod}: Pulling image "nginx:brokenpod"
  Warning  Failed     72s (x4 over 2m34s)  kubelet            spec.containers{sripod}: Failed to pull image "nginx:brokenpod": rpc error: code = NotFound desc = failed to pull and unpack image "docker.io/library/nginx:brokenpod": failed to resolve image: docker.io/library/nginx:brokenpod: not found
  Normal   BackOff    8s (x9 over 2m34s)   kubelet            spec.containers{sripod}: Back-off pulling image "nginx:brokenpod"
root@controlplane:~$
```

### e) Fix — correct the image tag
```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: sripod
  name: sripod
spec:
  containers:
  - image: nginx:latest
    name: sripod
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

```
root@controlplane:~$ k apply -f sripod.yaml
pod/sripod configured
root@controlplane:~$ k get pod
NAME     READY   STATUS    RESTARTS   AGE
sripod   1/1     Running   0          3m38s
root@controlplane:~$
```

## General debugging decision tree

1. `kubectl get pod` — check the STATUS column to identify the failure category (`Pending`, `ImagePullBackOff`, `CrashLoopBackOff`, `Error`, `OOMKilled`, etc.)
2. `kubectl describe pod <name>` — the Events section almost always shows the root cause directly (bad image, scheduling failure, failed probe, resource limit issue).
3. `kubectl logs <name>` (and `--previous` if the container already restarted) — shows application-level errors if the container did start and then crashed.
4. `kubectl get events --sort-by=.lastTimestamp` — cluster-wide event view if `describe` isn't enough.

**Common failure states:**
- `Pending` → scheduling issue (insufficient resources, taints/tolerations, node selectors)
- `ImagePullBackOff` / `ErrImagePull` → bad image name/tag, missing registry auth
- `CrashLoopBackOff` → container starts then exits (bad command, app crash, missing config/secret)
- `OOMKilled` → container exceeded memory limit (visible in `describe` under Last State)
