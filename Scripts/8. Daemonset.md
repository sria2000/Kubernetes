```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: anti-virus
spec:
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
        image: nginx
```
