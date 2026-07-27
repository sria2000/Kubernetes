# 11. How do you create a ReplicaSet with 2 replicas using a container image from Docker Hub?

Link: https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/

```yaml
# rs.yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: frontend
  labels:
    app: guestbook
    tier: frontend
spec:
  replicas: 2
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
      - name: nginx
        image: nginx:latest
```

```
root@controlplane:~$ k apply -f rs.yaml
replicaset.apps/frontend created
root@controlplane:~$ k get rs
NAME              DESIRED   CURRENT   READY   AGE
dep1-77bc6bd484   5         5         5       43m
dep2-7f984fdbcd   1         1         1       27m
frontend          2         2         2       22s
root@controlplane:~$
```

```
root@controlplane:~$ k describe rs frontend
Name:         frontend
Namespace:    default
Selector:     tier=frontend
Labels:       app=guestbook
              tier=frontend
Annotations:  <none>
Replicas:     2 current / 2 desired
Pods Status:  2 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:  tier=frontend
  Containers:
   nginx:
    Image:         nginx:latest
    Port:          <none>
    Host Port:     <none>
    Environment:   <none>
    Mounts:        <none>
  Volumes:         <none>
  Node-Selectors:  <none>
  Tolerations:     <none>
Events:
  Type    Reason            Age   From                   Message
  ----    ------            ----  ----                   -------
  Normal  SuccessfulCreate  29s   replicaset-controller  Created pod: frontend-cblzr
  Normal  SuccessfulCreate  29s   replicaset-controller  Created pod: frontend-qr7qg
root@controlplane:~$ k get pod -o wide | grep -i front
frontend-4nqdc          1/1     Running   0          38s   192.168.1.178   node01         <none>           <none>
frontend-wlslt          1/1     Running   0          38s   192.168.0.135   controlplane   <none>           <none>
root@controlplane:~$
```

**Note:** `nginx:latest` (no registry prefix) resolves to Docker Hub (`docker.io/library/nginx:latest`) by default. In production, prefer a pinned version tag over `:latest` for reproducibility and easier rollback.
