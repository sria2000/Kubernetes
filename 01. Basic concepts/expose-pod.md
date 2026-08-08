# Docker `EXPOSE` Instruction & Kubernetes Container Ports

## Overview

The `EXPOSE` instruction in a Dockerfile is used to **document** which network port(s) a containerized application listens on at runtime.

> **Important:** `EXPOSE` **does not publish** or make the port accessible from outside the container. It simply serves as documentation for anyone building or running the image.

---

## Official Documentation

### Docker

https://docs.docker.com/reference/dockerfile/#expose

### Kubernetes

https://kubernetes.io/docs/tutorials/stateless-application/expose-external-ip-address/#creating-a-service-for-an-application-running-in-five-pods

---

# What Does `EXPOSE` Do?

The `EXPOSE` instruction tells Docker that:

- The application inside the container listens on a specific port.
- The port is intended to be published if required.
- It acts as documentation between:
  - The person **building** the image.
  - The person **running** the container.

It **does not**:

- Publish the port.
- Open firewall rules.
- Allow access from outside the container.

---

# Syntax

```dockerfile
EXPOSE <port>
```

Example:

```dockerfile
EXPOSE 8080
```

or multiple ports

```dockerfile
EXPOSE 80
EXPOSE 443
```

---

# Example 1

Dockerfile

```dockerfile
FROM ubuntu

RUN apt-get update && apt-get install -y service

EXPOSE 1234

CMD ["service"]
```

Here Docker simply records that the application listens on port **1234**.

No port is published.

---

# Running a Container Without `-p`

```bash
docker run -d --name nginx nginx
```

Output

```text
090e4f2c6beb5b2cbb4ec4c2d4971b3f8576df8465a72361c350a6afaf039ee9
```

Verify

```bash
docker ps
```

Example

```text
CONTAINER ID   IMAGE     PORTS      NAMES
090e4f2c6beb   nginx     80/tcp     nginx
```

Notice:

```
80/tcp
```

This means:

- Container listens on port **80**
- Port **is NOT published**
- You cannot access it from your host

---

# Inspect the Container

Use:

```bash
docker inspect 090e4f2c6beb
```

Look for

```json
"ExposedPorts": {
    "80/tcp": {}
}
```

This confirms the image exposes port 80.

---

# Publishing a Port

To access the application from your host, publish the port using `-p`.

```bash
docker run -d \
--name nginx2 \
-p 80:80 \
nginx
```

Verify

```bash
docker ps
```

Output

```text
CONTAINER ID   IMAGE    PORTS
816f3d8d6901   nginx    0.0.0.0:80->80/tcp
```

Now Docker maps

```
Host Port      Container Port
-----------    --------------
80        -->       80
```

Your application is now accessible.

---

# Host vs Container Port

```
Browser
   |
localhost:80
   |
Docker Host
   |
Port Mapping (-p 80:80)
   |
Container Port 80
   |
Nginx
```

---

# Building Your Own Image

## Dockerfile

```dockerfile
FROM nginx:1.17-alpine

COPY index.html /usr/share/nginx/html
COPY nginx.conf /etc/nginx/

EXPOSE 9080

CMD ["nginx", "-g", "daemon off;"]
```

Notice

```
EXPOSE 9080
```

This tells Docker that Nginx will listen on **9080**.

---

## index.html

```html
<!DOCTYPE html>
<html>
<head>
    <title>Sriram's Nginx</title>
</head>

<body>
    <h1>Hello from Sriram's custom nginx image!</h1>
</body>
</html>
```

---

## nginx.conf

```nginx
events {
    worker_connections 1024;
}

http {

    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    sendfile on;
    keepalive_timeout 65;

    server {

        listen 9080;

        server_name localhost;

        location / {

            root /usr/share/nginx/html;

            index index.html index.htm;

        }

    }

}
```

---

# Build the Image

```bash
docker build -t my-nginx-image:1.0 .
```

---

# Run the Container

```bash
docker run -d \
-p 8080:9080 \
--name sri-nginx \
my-nginx-image:1.0
```

Docker performs the mapping

```
Host Port      Container Port
-----------    --------------
8080     -->      9080
```

---

# Verify

```bash
docker inspect sri-nginx
```

Look for

```json
"ExposedPorts": {
    "9080/tcp": {}
}
```

---

# Test

```bash
curl localhost:8080
```

Expected output

```html
<h1>Hello from Sriram's custom nginx image!</h1>
```

---

# EXPOSE vs Publish

| EXPOSE | Publish (`-p`) |
|---------|----------------|
| Documentation | Makes the port reachable |
| Happens during image build | Happens when the container runs |
| Inside Dockerfile | Docker run option |
| Does not expose externally | Exposes to the host |
| Metadata only | Creates actual port mapping |

---

# Kubernetes Equivalent

Docker uses

```dockerfile
EXPOSE 8080
```

Kubernetes uses

```yaml
ports:
- containerPort: 8080
```

The `containerPort` field is also **documentation**.

It does **not** expose the Pod outside the cluster.

To expose a Pod externally, you need a **Service** (ClusterIP, NodePort, LoadBalancer, or Ingress).

---

# Pod Example

Create

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

Save as

```text
pod-expose-port.yaml
```

---

# Deploy

```bash
kubectl apply -f pod-expose-port.yaml
```

Verify

```bash
kubectl get pods
```

Describe the Pod

```bash
kubectl describe pod nginx-pod
```

You'll see

```text
Port: 8080/TCP
```

---

# Kubernetes API Documentation

Learn more about the `ports` field

```bash
kubectl explain pod.spec.containers.ports
```

Example output

```
FIELD: ports <[]ContainerPort>

DESCRIPTION:

List of ports to expose from the container.
```

You can continue exploring:

```bash
kubectl explain pod.spec.containers.ports.containerPort
```

```bash
kubectl explain pod.spec.containers.ports.protocol
```

```bash
kubectl explain pod.spec.containers.ports.name
```

---

# Does `containerPort` Expose the Pod?

**No.**

Example

```
Pod
 |
 | containerPort:8080
 |
+-----------------------+
|        nginx          |
+-----------------------+
```

The Pod is still **not reachable** from outside.

You need a Service.

```
Browser
      |
      |
+-------------+
|  Service    |
+-------------+
       |
       |
+-------------+
|    Pod      |
| Port 8080   |
+-------------+
```

---

# Interview Questions

### Does `EXPOSE` publish a Docker port?

**No.**

It only documents the intended listening port.

---

### How do you publish a Docker port?

```bash
docker run -p 8080:80 nginx
```

---

### Is `EXPOSE` mandatory?

No.

The container works perfectly without it.

---

### What is the Kubernetes equivalent of `EXPOSE`?

```yaml
containerPort:
```

---

### Does `containerPort` expose a Pod externally?

No.

A Kubernetes **Service** is required.

---

### Why define `containerPort` then?

- Improves documentation.
- Helps other developers understand the application.
- Used by some Kubernetes tooling.
- Makes manifests easier to read and maintain.

---

# Summary

| Docker | Kubernetes |
|---------|------------|
| `EXPOSE 80` | `containerPort: 80` |
| Documentation only | Documentation only |
| Does not publish | Does not expose externally |
| Publish using `docker run -p` | Expose using a Service |
| Visible via `docker inspect` | Visible via `kubectl describe pod` |

---

## Key Takeaways

- `EXPOSE` is **documentation**, not port publishing.
- Use `docker run -p` to make a container accessible from the host.
- Verify exposed ports using `docker inspect`.
- In Kubernetes, `containerPort` is the equivalent of Docker's `EXPOSE`.
- To make a Pod accessible, create a **Service** (ClusterIP, NodePort, LoadBalancer, or Ingress), not just `containerPort`.
