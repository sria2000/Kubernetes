# Exposing a Pod and Understanding `containerPort`

## Objective

Learn the difference between:

- `containerPort`
- The actual application listening port
- `kubectl port-forward`

One of the most common misconceptions in Kubernetes is believing that `containerPort` makes an application listen on that port. It does **not**.

---

## Step 1: Create a Pod

Create a file named `expose.yaml`.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
  - image: nginx
    name: democontainer
    ports:
    - containerPort: 8080
```

Apply the manifest:

```bash
kubectl apply -f expose.yaml
```

Verify:

```bash
kubectl get pods
```

Example:

```text
NAME        READY   STATUS    RESTARTS   AGE
nginx-pod   1/1     Running   0          20s
```

---

## Step 2: Describe the Pod

```bash
kubectl describe pod nginx-pod
```

Notice:

```text
Containers:
  democontainer:
    Image: nginx
    Port: 8080/TCP
```

Kubernetes displays **8080** because that is what was declared in the Pod manifest.

This **does not** mean Nginx is listening on port **8080**.

---

## Step 3: Check the Meaning of `containerPort`

```bash
kubectl explain pod.spec.containers.ports
```

You will see that `containerPort` is simply the port that the container exposes.

It is **metadata** and **does not configure the application**.

---

## Step 4: Verify What Nginx Is Actually Listening On

Enter the Pod:

```bash
kubectl exec -it nginx-pod -- /bin/bash
```

Run:

```bash
nginx -T | grep listen
```

Output:

```text
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful

listen 80;
listen [::]:80;
```

Notice:

- YAML says **8080**
- Nginx listens on **80**

The application determines the listening port—not Kubernetes.

---

## Step 5: Try Accessing Port 8080

From the node:

```bash
curl http://localhost:8080
```

Output:

```text
curl: (7) Failed to connect to localhost port 8080
```

This fails because nothing is listening on port **8080**.

---

## Step 6: Forward the Correct Port

Forward your local port **8080** to the Pod's actual listening port **80**.

```bash
kubectl port-forward pod/nginx-pod 8080:80
```

Output:

```text
Forwarding from 127.0.0.1:8080 -> 80
Forwarding from [::1]:8080 -> 80
```

Open another terminal and test:

```bash
curl http://localhost:8080
```

Output:

```html
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
...
```

Success!

---

# Why Did This Work?

```
Local Machine
     │
     ▼
localhost:8080
     │
     ▼
kubectl port-forward
     │
     ▼
Pod Port 80
     │
     ▼
Nginx
```

Even though the Pod manifest declared:

```yaml
containerPort: 8080
```

the application was actually listening on:

```text
80
```

So we forwarded:

```text
localhost:8080  --->  Pod:80
```

---

# Understanding `containerPort`

This field:

```yaml
ports:
- containerPort: 8080
```

does **not**:

- Open a port
- Configure Nginx
- Change the application's listening port
- Publish the Pod on the network

It only provides metadata for Kubernetes and for anyone reading the manifest.

---

# How to Make Nginx Listen on 8080

You must modify the Nginx configuration, for example:

```nginx
server {
    listen 8080;
}
```

Simply changing `containerPort` does **not** reconfigure Nginx.

---

# Summary

| Component | Port |
|-----------|------|
| `containerPort` in YAML | 8080 |
| Actual Nginx listening port | 80 |
| `curl localhost:8080` (before port-forward) | ❌ Failed |
| `kubectl port-forward pod/nginx-pod 8080:80` | ✅ Success |
| `curl localhost:8080` (after port-forward) | ✅ Nginx Welcome Page |

---

# Key Takeaways

- `containerPort` is **metadata only**.
- Kubernetes does **not** configure the application.
- The application determines which port it listens on.
- `kubectl port-forward` must forward traffic to the application's actual listening port.
- For the official Nginx image, the default listening port is **80**.

---

# CKA Exam Tip

Remember this one-liner:

> **`containerPort` documents the intended port, but the application itself decides which port it actually listens on.**
