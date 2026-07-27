```
apiVersion: v1
kind: Pod
metadata:
  name: largepod
spec:
  nodeSelector:
    size: Large
  containers:
  - name: nginx-pod
    image: nginx
```
