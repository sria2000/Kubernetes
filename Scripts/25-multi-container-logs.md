# 15. How do you check logs for a specific container in a multi-container Pod?

```yaml
# multi.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80
  - name: redis
    image: redis:latest
```

```
root@controlplane:~$ k apply -f multi.yaml
pod/nginx created
root@controlplane:~$ k get pods
NAME    READY   STATUS              RESTARTS   AGE
nginx   0/2     ContainerCreating   0          4s
```
Shows 2/2 — confirms this is a multi-container pod.

```
root@controlplane:~$ k get pods
NAME    READY   STATUS    RESTARTS   AGE
nginx   2/2     Running   0          5s
root@controlplane:~$
```

```
root@controlplane:~$ k describe pod nginx
Name:             nginx
Namespace:        default
Priority:         0
Service Account:  default
Node:             node01/172.30.2.2
Start Time:       Mon, 27 Jul 2026 19:56:25 +0000
Labels:           <none>
Annotations:      <none>
Status:           Running
IP:               192.168.1.134
IPs:
  IP:  192.168.1.134
Containers:
  nginx:
    Container ID:   containerd://ce70852d153d863f256fa338376800d940e0682e926569e129bbe14dcd2f740c
    Image:          nginx:1.14.2
    Image ID:       docker.io/library/nginx@sha256:f7988fb6c02e0ce69257d9bd9cf37ae20a60f1df7563c3a2a6abe24160306b8d
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Mon, 27 Jul 2026 19:56:26 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-nftn8 (ro)
  redis:
    Container ID:   containerd://3891cb1bad1c63128ca5150d86811866a0ca30577beea45183fd433bcb729ad4
    Image:          redis:latest
    Image ID:       docker.io/library/redis@sha256:c88d347edef6249a6d2293f926f1eeb48bd40c57cbcd02c07f52e7f1fd2cb46b
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Mon, 27 Jul 2026 19:56:29 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-nftn8 (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True
  Initialized                 True
  Ready                       True
  ContainersReady             True
  PodScheduled                True
Volumes:
  kube-api-access-nftn8:
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
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  22s   default-scheduler  Successfully assigned default/nginx to node01
  Normal  Pulled     21s   kubelet            spec.containers{nginx}: Container image "nginx:1.14.2" already present on machine and can be accessed by the pod
  Normal  Created    21s   kubelet            spec.containers{nginx}: Container created
  Normal  Started    21s   kubelet            spec.containers{nginx}: Container started
  Normal  Pulling    21s   kubelet            spec.containers{redis}: Pulling image "redis:latest"
  Normal  Pulled     18s   kubelet            spec.containers{redis}: Successfully pulled image "redis:latest" in 3.544s (3.544s including waiting). Image size: 54289490 bytes.
  Normal  Created    18s   kubelet            spec.containers{redis}: Container created
  Normal  Started    18s   kubelet            spec.containers{redis}: Container started
root@controlplane:~$
```

### Plain `logs` defaults to the first container
```
root@controlplane:~$ k logs nginx
Defaulted container "nginx" out of: nginx, redis
root@controlplane:~$
```

### Target a specific container with `-c`
```
root@controlplane:~$ k logs nginx -c redis
Starting Redis Server
1:C 27 Jul 2026 19:56:29.827 * oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
1:C 27 Jul 2026 19:56:29.827 * Redis version=8.8.1, bits=64, commit=00000000, modified=1, pid=1, just started
1:C 27 Jul 2026 19:56:29.827 * Configuration loaded
1:M 27 Jul 2026 19:56:29.828 * Increased maximum number of open files to 10032 (it was originally set to 1024).
1:M 27 Jul 2026 19:56:29.828 * monotonic clock: POSIX clock_gettime
1:M 27 Jul 2026 19:56:29.829 * Running mode=standalone, port=6379.
...
1:M 27 Jul 2026 19:56:29.842 * Ready to accept connections tcp
1:M 27 Jul 2026 19:56:29.842 # WARNING: Redis does not require authentication and is not protected by network restrictions. Redis will accept connections from any IP address on any network interface.
root@controlplane:~$ k logs nginx -c nginx
root@controlplane:~$
```
(Empty output for nginx is expected — access/error logs only appear on incoming requests, and none were made.)

**Tip:** To get logs from all containers in a pod at once:
```
k logs nginx --all-containers=true
```
