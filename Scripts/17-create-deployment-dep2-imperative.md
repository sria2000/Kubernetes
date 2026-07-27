# 7. How do you create a Deployment named dep2 using the imperative method with replica count set to 1?

Link: https://kubernetes.io/docs/reference/kubectl/generated/kubectl_create/kubectl_create_deployment/

```
root@controlplane:~$ kubectl create deployment dep2 --image=nginx --replicas=1
deployment.apps/dep2 created
root@controlplane:~$
root@controlplane:~$ k get deployment
NAME   READY   UP-TO-DATE   AVAILABLE   AGE
dep1   5/5     5            5           16m
dep2   1/1     1            1           56s
root@controlplane:~$
```

```
root@controlplane:~$ k describe deployment dep2
Name:                   dep2
Namespace:              default
CreationTimestamp:      Mon, 27 Jul 2026 16:29:54 +0000
Labels:                 app=dep2
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=dep2
Replicas:               1 desired | 1 updated | 1 total | 1 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=dep2
  Containers:
   nginx:
    Image:         nginx
    Port:          <none>
    Host Port:     <none>
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
NewReplicaSet:   dep2-7f984fdbcd (1/1 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  61s   deployment-controller  Scaled up replica set dep2-7f984fdbcd from 0 to 1
root@controlplane:~$
```
