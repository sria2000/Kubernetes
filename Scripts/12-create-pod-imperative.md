# 2. How do you create a Pod directly using a command without using a YAML file?

Document: https://kubernetes.io/docs/reference/kubectl/generated/kubectl_run/

```
root@controlplane:~$ kubectl run hazelcast --image=hazelcast/hazelcast --port=5701
pod/hazelcast created
root@controlplane:~$
root@controlplane:~$ k get pod
NAME        READY   STATUS    RESTARTS   AGE
hazelcast   1/1     Running   0          72s
nginx       1/1     Running   0          6m41s
root@controlplane:~$
```

```
root@controlplane:~$ k describe pod hazelcast
Name:             hazelcast
Namespace:        default
Priority:         0
Service Account:  default
Node:             node01/172.30.2.2
Start Time:       Mon, 27 Jul 2026 16:09:35 +0000
Labels:           run=hazelcast
Annotations:      <none>
Status:           Running
IP:               192.168.1.197
IPs:
  IP:  192.168.1.197
Containers:
  hazelcast:
    Container ID:   containerd://d64c421a369eab3fc592718e053b50064ff519ded533a3f681ed6644d05f769d
    Image:          hazelcast/hazelcast
    Image ID:       docker.io/hazelcast/hazelcast@sha256:f086bf0ecb23fcb2e9a516e337e51018156b89df41543ab768b8468ab281ad67
    Port:           5701/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Mon, 27 Jul 2026 16:10:32 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-xcbwv (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True
  Initialized                 True
  Ready                       True
  ContainersReady             True
  PodScheduled                True
Volumes:
  kube-api-access-xcbwv:
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
  Type    Reason     Age    From               Message
  ----    ------     ----   ----               -------
  Normal  Scheduled  2m27s  default-scheduler  Successfully assigned default/hazelcast to node01
  Normal  Pulling    2m26s  kubelet            spec.containers{hazelcast}: Pulling image "hazelcast/hazelcast"
  Normal  Pulled     90s    kubelet            spec.containers{hazelcast}: Successfully pulled image "hazelcast/hazelcast" in 55.503s (55.503s including waiting). Image size: 618698769 bytes.
  Normal  Created    90s    kubelet            spec.containers{hazelcast}: Container created
  Normal  Started    90s    kubelet            spec.containers{hazelcast}: Container started
root@controlplane:~$
```
