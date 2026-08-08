# Kubernetes Service Selectors - Automatically Registering Service Endpoints

## Introduction

In the previous lab, we manually created an **Endpoints** object and associated it with a Service.

### Manual Workflow

```
Service
   |
   |
Endpoints (Manually Created)
   |
   |
Backend Pods
```

Files used:

- **service.yaml** – Defines the Service, Port and TargetPort.
- **endpoint.yaml** – References the Service and manually specifies the Pod IP addresses.

### Problem with Manual Endpoints

Although this works, it has a major drawback.

- Pod IP addresses are **dynamic**.
- Every time a Pod is recreated, it receives a new IP address.
- The Endpoints object must be updated manually.

This approach is difficult to maintain in production.

---

# The Better Solution - Service Selectors

Instead of manually creating Endpoints, Kubernetes can automatically discover Pods using **Labels** and **Selectors**.

```
          Service
      Selector: app=backend
               |
               |
        -----------------
        |               |
   Backend Pod 1   Backend Pod 2
    app=backend     app=backend
```

Whenever a Pod matches the selector, Kubernetes automatically adds it to the Service Endpoints.

No manual Endpoints object is required.

---

# How It Works

```
Pod Labels
      |
      |
Service Selector
      |
      |
Automatic Endpoints
      |
      |
Traffic routed to Pods
```

Kubernetes continuously watches for Pods whose labels match the Service selector.

---

# Base Service Manifest

Create a basic Service.

**service.yaml**

```yaml
apiVersion: v1
kind: Service

metadata:
  name: simple-service

spec:
  ports:
    - port: 80
      targetPort: 80
```

At this point, the Service has **no selector**, so it cannot discover any Pods automatically.

---

# Add a Selector

Modify the Service to include a selector.

```yaml
apiVersion: v1
kind: Service

metadata:
  name: simple-service

spec:
  selector:
    app: backend

  ports:
    - port: 80
      targetPort: 80
```

Apply the Service.

```bash
kubectl apply -f service.yaml
```

Describe the Service.

```bash
kubectl describe service simple-service
```

Output:

```
Selector:
app=backend

Endpoints:
<none>
```

There are no matching Pods yet.

---

# Create Backend Pods

Create two backend Pods.

```bash
kubectl run backend-pod-1 --image=nginx

kubectl run backend-pod-2 --image=nginx
```

Verify the Pods.

```bash
kubectl get pods
```

---

# View Existing Labels

Display Pod labels.

```bash
kubectl get pods --show-labels
```

Example:

```
NAME            LABELS

backend-pod-1   run=backend-pod-1

backend-pod-2   run=backend-pod-2
```

Notice that neither Pod has the label:

```
app=backend
```

Therefore, the Service still has no Endpoints.

---

# Add Labels to the Pods

Label the first Pod.

```bash
kubectl label pod backend-pod-1 app=backend
```

Label the second Pod.

```bash
kubectl label pod backend-pod-2 app=backend
```

Verify the labels.

```bash
kubectl get pods --show-labels
```

Example:

```
NAME            LABELS

backend-pod-1   app=backend,run=backend-pod-1

backend-pod-2   app=backend,run=backend-pod-2
```

---

# Service Automatically Registers Endpoints

Describe the Service again.

```bash
kubectl describe service simple-service
```

Output:

```
Selector:
app=backend

Endpoints:
192.168.1.170:80,
192.168.1.253:80
```

Notice that **no Endpoints object was created manually**.

Kubernetes automatically registered both Pods because their labels matched the Service selector.

---

# Verify Pod IPs

```bash
kubectl get pods -o wide
```

Example:

```
NAME            IP

backend-pod-1   192.168.1.170

backend-pod-2   192.168.1.253
```

These are exactly the IP addresses listed under the Service Endpoints.

---

# Automatic Endpoint Updates

One of the biggest advantages of Selectors is that Endpoints are updated automatically.

Suppose we remove the label from one Pod.

```bash
kubectl label pod backend-pod-1 app-
```

Verify the labels.

```bash
kubectl get pods --show-labels
```

Output:

```
backend-pod-1
run=backend-pod-1

backend-pod-2
app=backend,run=backend-pod-2
```

Now describe the Service again.

```bash
kubectl describe service simple-service
```

Output:

```
Endpoints:
192.168.1.253:80
```

The first Pod immediately disappears from the Service Endpoints.

No manual changes were required.

---

# Dynamic Registration

If you add the label back:

```bash
kubectl label pod backend-pod-1 app=backend
```

The Pod is automatically added back to the Service.

Likewise:

- Creating a new Pod with `app=backend` automatically registers it.
- Deleting a Pod automatically removes it from the Endpoints.
- Changing labels immediately updates the Endpoints.

Everything happens automatically.

---

# Manual Endpoints vs Selectors

| Manual Endpoints | Service Selectors |
|------------------|-------------------|
| Requires an Endpoints object | No Endpoints object required |
| Pod IPs must be maintained manually | Kubernetes discovers Pods automatically |
| Not suitable for dynamic environments | Ideal for production workloads |
| Easy to make mistakes | Self-managing |
| Does not scale well | Scales automatically |

---

# Service Discovery Flow

```
             Service
      Selector: app=backend
               |
               |
        Kubernetes Controller
               |
      -------------------------
      |                       |
Backend Pod 1           Backend Pod 2
app=backend             app=backend
      |
      |
Automatic Endpoints
```

---

# Useful Commands

Create the Service

```bash
kubectl apply -f service.yaml
```

Create Pods

```bash
kubectl run backend-pod-1 --image=nginx

kubectl run backend-pod-2 --image=nginx
```

View Labels

```bash
kubectl get pods --show-labels
```

Add a Label

```bash
kubectl label pod backend-pod-1 app=backend
```

Remove a Label

```bash
kubectl label pod backend-pod-1 app-
```

View Service Details

```bash
kubectl describe service simple-service
```

View Pod IPs

```bash
kubectl get pods -o wide
```

---

# Interview Questions

### Why do we use Selectors in a Service?

Selectors allow Kubernetes to automatically discover Pods based on their labels and register them as Service Endpoints.

---

### Does Kubernetes create the Endpoints object automatically?

Yes.

When a Service has a selector, Kubernetes automatically creates and maintains the associated Endpoints (or EndpointSlices in newer Kubernetes versions).

---

### What happens if a Pod is deleted?

Its IP address is automatically removed from the Service Endpoints.

---

### What happens if a new Pod with the correct label is created?

Kubernetes automatically adds it to the Service Endpoints.

---

### What happens if the label is removed from a Pod?

The Pod immediately stops receiving traffic because it is removed from the Service Endpoints.

---

# Key Takeaways

- Services identify Pods using **Labels** and **Selectors**.
- No manual Endpoints object is needed when using Selectors.
- Kubernetes automatically creates and updates the Service Endpoints.
- Pods are added or removed from the Service dynamically as labels change.
- This automatic registration makes Services highly scalable and suitable for production environments.
