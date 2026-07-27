```
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: nginx
    env: dev
  name: nginx

spec:
  containers:
  - image: nginx
    name: nginx
```
