# 5. How do you verify and confirm that two new Pods are created after scaling the Deployment?

### a) Check deployment first
```
root@controlplane:~$ k get deployment
NAME   READY   UP-TO-DATE   AVAILABLE   AGE
dep1   5/5     5            5           3m2s
root@controlplane:~$
```

### b) Check pods
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

### c) Describe replica and confirm 5 total
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

**Tip:** `kubectl rollout status deployment/dep1` is another clean way to confirm scaling has completed successfully.
