# 17. How do you identify image pull or scheduling issues in Kubernetes?

## Part A: Image pull issue

```yaml
# pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:9.9
    ports:
    - containerPort: 80
```

```
root@controlplane:~$ k apply -f pod.yaml
pod/nginx created
root@controlplane:~$ k get pods
NAME    READY   STATUS              RESTARTS   AGE
nginx   0/1     ContainerCreating   0          3s
root@controlplane:~$ k get pods
NAME    READY   STATUS         RESTARTS   AGE
nginx   0/1     ErrImagePull   0          7s
root@controlplane:~$
```

### Describe
```
root@controlplane:~$ k describe pod nginx
Name:             nginx
Namespace:        default
Priority:         0
Service Account:  default
Node:             node01/172.30.2.2
Start Time:       Mon, 27 Jul 2026 20:04:11 +0000
Labels:           <none>
Annotations:      <none>
Status:           Pending
IP:               192.168.1.160
IPs:
  IP:  192.168.1.160
Containers:
  nginx:
    Container ID:
    Image:          nginx:9.9
    Image ID:
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Waiting
      Reason:       ErrImagePull
    Ready:          False
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-4skcc (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True
  Initialized                 True
  Ready                       False
  ContainersReady             False
  PodScheduled                True
Volumes:
  kube-api-access-4skcc:
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
  Normal   Scheduled  33s                default-scheduler  Successfully assigned default/nginx to node01
  Normal   Pulling    19s (x2 over 32s)  kubelet            spec.containers{nginx}: Pulling image "nginx:9.9"
  Warning  Failed     17s (x2 over 30s)  kubelet            spec.containers{nginx}: Failed to pull image "nginx:9.9": rpc error: code = NotFound desc = failed to pull and unpack image "docker.io/library/nginx:9.9": failed to resolve image: docker.io/library/nginx:9.9: not found
  Warning  Failed     17s (x2 over 30s)  kubelet            spec.containers{nginx}: Error: ErrImagePull
  Normal   BackOff    6s (x2 over 30s)   kubelet            spec.containers{nginx}: Back-off pulling image "nginx:9.9"
  Warning  Failed     6s (x2 over 30s)   kubelet            spec.containers{nginx}: Error: ImagePullBackOff
root@controlplane:~$
```

### Logs (not useful for this state)
```
root@controlplane:~$ k logs nginx
Error from server (BadRequest): container "nginx" in pod "nginx" is waiting to start: trying and failing to pull image
root@controlplane:~$
```
Logs and describe both point to the image as the problem.

### Fix — correct the image tag
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80
```

```
root@controlplane:~$ k apply -f pod.yaml
pod/nginx configured
root@controlplane:~$ k get pods
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          2m9s
root@controlplane:~$
```

## Part B: Scheduling (unschedulable) issue

Create a pod requesting an impossible amount of CPU (request must equal or be under any limit set, so both are specified to avoid an admission rejection):

```yaml
# unsched.yaml
apiVersion: v1
kind: Pod
metadata:
  name: cpu-resource-constraint
spec:
  containers:
  - name: nginx
    image: nginx
    resources:
      requests:
        cpu: "1000"
      limits:
        cpu: "1000"
```

```
root@controlplane:~$ k apply -f unsched.yaml
pod/cpu-resource-constraint created
root@controlplane:~$ k describe pod cpu-resource-constraint | tail -4
Events:
  Type     Reason            Age   From               Message
  ----     ------            ----  ----               -------
  Warning  FailedScheduling  59s   default-scheduler  0/2 nodes are available: 2 Insufficient cpu. no new claims to deallocate, preemption: 0/2 nodes are available: 2 Preemption is not helpful for scheduling.
root@controlplane:~$
```

## Summary: distinguishing the two failure types

| Failure type | Signal in `get pod` | Signal in `describe pod` |
|---|---|---|
| **Image pull issue** | `ErrImagePull` → `ImagePullBackOff` | `Scheduled` event present, then `Pulling → Failed → BackOff` |
| **Scheduling issue** | Stuck `Pending` | No `Scheduled` event; `FailedScheduling` warning with reason (e.g. `Insufficient cpu`) |

**Interview one-liner:** "I check `kubectl get pod` for the status first — `ImagePullBackOff` means the image couldn't be pulled, while a pod stuck in `Pending` with no node assigned points to a scheduling problem. Then `kubectl describe pod` confirms it: if there's a `Scheduled` event followed by pull failures, it's an image issue; if there's no `Scheduled` event and instead a `FailedScheduling` warning, it's a scheduling issue — usually insufficient resources, taints/tolerations, or affinity rules."

**Other common scheduling failure causes worth knowing:**
- Insufficient resources (CPU/memory) — demonstrated above
- Node taints without matching tolerations
- `nodeSelector` / `nodeAffinity` mismatch — no node has the required label
- `PodAntiAffinity` conflicts or unsatisfiable topology spread constraints

**Note on LimitRange vs Pod resource requests:** A `LimitRange` is a namespace-level admission-control object that sets default/min/max CPU-memory bounds for containers in that namespace. If a pod's request/limit violates it, the pod is **rejected at `kubectl apply` time** (never created) — this is a different failure mode from a genuine scheduling failure, where the pod *is* created but the scheduler can't place it on any node.
