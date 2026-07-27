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
<img width="254" height="201" alt="image" src="https://github.com/user-attachments/assets/a74f8fe4-92c9-4574-b556-6fe8c4a66649" />
