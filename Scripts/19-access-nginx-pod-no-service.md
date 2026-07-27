# 9. How do you access the nginx web page of a Pod running inside the dep1 Deployment without using a Service?

```
root@controlplane:~$ k get pods -o wide | grep dep1
dep1-77bc6bd484-29p5j   1/1     Running   0          33m     192.168.1.233   node01         <none>           <none>
dep1-77bc6bd484-c9gpw   1/1     Running   0          33m     192.168.0.144   controlplane   <none>           <none>
dep1-77bc6bd484-ff8f4   1/1     Running   0          31m     192.168.1.198   node01         <none>           <none>
dep1-77bc6bd484-h47ww   1/1     Running   0          33m     192.168.1.13    node01         <none>           <none>
dep1-77bc6bd484-jmz26   1/1     Running   0          31m     192.168.0.193   controlplane   <none>           <none>
root@controlplane:~$
```

```
root@controlplane:~$ k describe pod dep1-77bc6bd484-29p5j
Name:             dep1-77bc6bd484-29p5j
Namespace:        default
Priority:         0
Service Account:  default
Node:             node01/172.30.2.2
Start Time:       Mon, 27 Jul 2026 16:14:45 +0000
Labels:           app=nginx
                  pod-template-hash=77bc6bd484
Annotations:      <none>
Status:           Running
IP:               192.168.1.233
IPs:
  IP:           192.168.1.233
Controlled By:  ReplicaSet/dep1-77bc6bd484
Containers:
  nginx:
    Container ID:   containerd://8f9c19d3d96bc8a9fb083a19727fdb3edf2118a968aa089d3b2258370f5c4282
    Image:          nginx:1.14.2
    Image ID:       docker.io/library/nginx@sha256:f7988fb6c02e0ce69257d9bd9cf37ae20a60f1df7563c3a2a6abe24160306b8d
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Mon, 27 Jul 2026 16:14:46 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-7jk6n (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True
  Initialized                 True
  Ready                       True
  ContainersReady             True
  PodScheduled                True
Volumes:
  kube-api-access-7jk6n:
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
  Normal  Scheduled  33m   default-scheduler  Successfully assigned default/dep1-77bc6bd484-29p5j to node01
  Normal  Pulled     33m   kubelet            spec.containers{nginx}: Container image "nginx:1.14.2" already present on machine and can be accessed by the pod
  Normal  Created    33m   kubelet            spec.containers{nginx}: Container created
  Normal  Started    33m   kubelet            spec.containers{nginx}: Container started
root@controlplane:~$ curl 192.168.1.233:80
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
root@controlplane:~$
```

**Key concept:** Pods get a directly routable cluster IP (flat pod network, no NAT between pods), so any pod is reachable by `<pod-IP>:<port>` from any node/pod in the cluster without a Service. A Service just adds a stable virtual IP/DNS name and load balancing on top of a changing set of pod IPs — it's not a networking requirement. Note this only works from *inside* the cluster; pod IPs aren't reachable from outside.

`Controlled By: ReplicaSet/dep1-77bc6bd484` confirms this pod genuinely belongs to the `dep1` Deployment.
