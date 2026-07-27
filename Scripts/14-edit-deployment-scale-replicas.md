# 4. How do you edit the existing Deployment dep1 to increase the replicas from 3 to 5?

```yaml
# deploy.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: dep1
  labels:
    app: nginx
spec:
  replicas: 5
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

```
root@controlplane:~$ k apply -f deploy.yaml
deployment.apps/dep1 configured
root@controlplane:~$ k get deployment
NAME   READY   UP-TO-DATE   AVAILABLE   AGE
dep1   5/5     5            5           3m2s
root@controlplane:~$
```

```
root@controlplane:~$ k get pods
NAME                    READY   STATUS    RESTARTS   AGE
dep1-77bc6bd484-29p5j   1/1     Running   0          3m19s
dep1-77bc6bd484-c9gpw   1/1     Running   0          3m19s
dep1-77bc6bd484-ff8f4   1/1     Running   0          89s
dep1-77bc6bd484-h47ww   1/1     Running   0          3m19s
dep1-77bc6bd484-jmz26   1/1     Running   0          89s
root@controlplane:~$
```

```
root@controlplane:~$ k describe deployment dep1
Name:                   dep1
Namespace:              default
CreationTimestamp:      Mon, 27 Jul 2026 16:14:45 +0000
Labels:                 app=nginx
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=nginx
Replicas:               5 desired | 5 updated | 5 total | 5 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=nginx
  Containers:
   nginx:
    Image:         nginx:1.14.2
    Port:          80/TCP
    Host Port:     0/TCP
    Environment:   <none>
    Mounts:        <none>
  Volumes:         <none>
  Node-Selectors:  <none>
  Tolerations:     <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Progressing    True    NewReplicaSetAvailable
  Available      True    MinimumReplicasAvailable
OldReplicaSets:  <none>
NewReplicaSet:   dep1-77bc6bd484 (5/5 replicas created)
Events:
  Type    Reason             Age    From                   Message
  ----    ------             ----   ----                   -------
  Normal  ScalingReplicaSet  3m43s  deployment-controller  Scaled up replica set dep1-77bc6bd484 from 0 to 3
  Normal  ScalingReplicaSet  113s   deployment-controller  Scaled up replica set dep1-77bc6bd484 from 3 to 5
root@controlplane:~$
```

**Note:** Editing via re-applying the YAML is one valid method. Two other common approaches:
- `kubectl edit deployment dep1` (edits the live object interactively)
- `kubectl scale deployment dep1 --replicas=5` (imperative, no YAML needed)
