# 16. How do you check Kubernetes events in a namespace?

### Create a namespace
```
root@controlplane:~$ k create ns test-namespace
namespace/test-namespace created
root@controlplane:~$
```

### Create a pod in that namespace
```
root@controlplane:~$ k run nginx --image=nginx:latest -n test-namespace
pod/nginx created
root@controlplane:~$
```

### Confirm the pod exists in the namespace
```
root@controlplane:~$ k get pod -n test-namespace
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          14s
```

### Check events
```
root@controlplane:~$ k events -n test-namespace
LAST SEEN   TYPE     REASON      OBJECT      MESSAGE
17s         Normal   Scheduled   Pod/nginx   Successfully assigned test-namespace/nginx to node01
17s         Normal   Pulling     Pod/nginx   Pulling image "nginx:latest"
12s         Normal   Pulled      Pod/nginx   Successfully pulled image "nginx:latest" in 4.799s (4.799s including waiting). Image size: 63132183 bytes.
12s         Normal   Created     Pod/nginx   Container created
12s         Normal   Started     Pod/nginx   Container started
root@controlplane:~$
```

**Notes:**
- `kubectl events` is the newer, cleaner command. The older equivalent is:
  ```
  kubectl get events -n test-namespace --sort-by=.lastTimestamp
  ```
- To see events across all namespaces:
  ```
  kubectl events -A
  ```
  or the older `kubectl get events --all-namespaces`.
