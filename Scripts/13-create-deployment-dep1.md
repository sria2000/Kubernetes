# 3. How do you create a Deployment named dep1 with nginx image and replicas set to 3?

Link: https://kubernetes.io/docs/concepts/workloads/controllers/deployment/

```yaml
# deploy.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: dep1
  labels:
    app: nginx
spec:
  replicas: 3
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
deployment.apps/dep1 created
root@controlplane:~$
root@controlplane:~$ k get deployment
NAME   READY   UP-TO-DATE   AVAILABLE   AGE
dep1   3/3     3            3           11s
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
Replicas:               3 desired | 3 updated | 3 total | 3 available | 0 unavailable
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
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   dep1-77bc6bd484 (3/3 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  49s   deployment-controller  Scaled up replica set dep1-77bc6bd484 from 0 to 3
root@controlplane:~$
```
