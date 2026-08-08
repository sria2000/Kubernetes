# Kubernetes Resource Creation Methods

## Basic K8s Resource Types

- Pod
- Deployment
- Service
- ConfigMap
- Secret
- PersistentVolume
- PersistentVolumeClaim
- Ingress
- Namespace

In Kubernetes, there are two ways to create resources:

1. **Imperative Approach**
2. **Declarative Approach**

---

## 1. Imperative Approach

### Overview

In the imperative approach, we directly give Kubernetes a command to create a resource.

```bash
kubectl run <pod-name> --image=<image-name>
```

Kubernetes immediately creates the resource based on the command provided.

### Create a Pod Using the Imperative Method

Check existing pods:

```bash
root@k8s-master:/etc/kubernetes# k get pods

NAME   READY   STATUS    RESTARTS   AGE
poda   1/1     Running   0          3h17m
```

Create a new nginx pod:

```bash
root@k8s-master:/etc/kubernetes# k run sri-imperative-pod --image=nginx

pod/sri-imperative-pod created
```

Verify the pod:

```bash
root@k8s-master:/etc/kubernetes# k get pods

NAME                 READY   STATUS    RESTARTS   AGE
poda                 1/1     Running   0          3h18m
sri-imperative-pod   1/1     Running   0          47s
```

### Describe the Pod

To get detailed information about the pod:

```bash
root@k8s-master:/etc/kubernetes# k describe pod sri-imperative-pod
```

Example output:

```
Name:             sri-imperative-pod
Namespace:        default
```

The `describe` command provides information such as:

- Pod name
- Namespace
- Node placement
- Container details
- Image details
- Events
- Status information

### Summary of Imperative Commands

**1. Create a Pod from Nginx Image**

```bash
kubectl run nginx --image=nginx
kubectl get pods
```

**2. Create a Pod from Nginx Image with Dry Run**

```bash
kubectl run nginx2 --image=nginx --dry-run=client
kubectl get pods
```
> Object is not created — command only validates.

**3. Create a Pod from Nginx Image with YAML Output**

```bash
kubectl run nginx2 --image=nginx -o yaml
```
> Prints the whole YAML after creating the pod.

**4. Generate a Manifest File (key command)**

```bash
kubectl run nginx4 --image=nginx --dry-run=client -o yaml

kubectl run nginx4 --image=nginx --dry-run=client -o yaml > pod-custom.yaml
```

**5. Delete All Pods**

```bash
kubectl delete pod --all
```

---

## 2. Declarative Approach

### Overview

In the declarative approach, we define the desired Kubernetes state in a YAML file.

Kubernetes compares the current state with the desired state and makes the required changes.

```bash
kubectl apply -f <yaml-file>
```

### Advantages

- Infrastructure can be stored as code
- YAML files can be version controlled using Git
- Easy to reproduce environments
- Kubernetes maintains the desired state

### Creating a Kubernetes Pod Using YAML

#### Step 1: Create YAML File

```bash
vi declarative.yaml
```

Basic Kubernetes YAML structure:

```yaml
apiVersion: <API group where resource exists>
kind: <Resource type>
metadata:
  name: <resource-name>
  labels:
    app: <label-name>

spec:
  containers:
  - name: <container-name>
    image: <container-image>
    ports:
    - containerPort: <application-port>
```

Useful commands to explore API resources:

```bash
kubectl api-resources
kubectl api-resources | grep -i pod
```

**In the YAML:**

| Field | Description |
|---|---|
| `apiVersion` | Check `v1` |
| `kind` | e.g. `Pod` |
| `metadata` | Name of the pod |
| `spec.containers.name` | Name of the container |
| `spec.containers.image` | Image needed |

#### Step 2: Create Pod Definition

Example YAML:

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: sri-declarative
  labels:
    app: sri-http-label

spec:
  containers:
  - name: sri-container
    image: nginx
    ports:
    - containerPort: 80
```

#### Step 3: Apply YAML Configuration

Create the pod using the YAML file:

```bash
root@ubuntu:~$ kubectl apply -f declarative.yaml

pod/sri-declarative created
```

#### Verify Pod Creation

Check all pods:

```bash
root@k8s-master:~# k get pods

NAME                 READY   STATUS    RESTARTS   AGE
poda                 1/1     Running   0          3h23m
sri-declarative      1/1     Running   0          5s
sri-http             1/1     Running   0          3m29s
sri-imperative-pod   1/1     Running   0          5m40s
```

The pod `sri-declarative` was created using the YAML definition.

---

## Access Nginx Application Using Port Forwarding

Pods are normally accessible only inside the Kubernetes cluster.

Use `kubectl port-forward` to expose the pod locally.

Run port forwarding in the background:

```bash
nohup kubectl port-forward pod/sri-declarative 8080:80 > port-forward.log 2>&1 &
```

### Explanation

| Parameter | Description |
|---|---|
| `pod/sri-declarative` | Pod to expose |
| `8080` | Local machine port |
| `80` | Container port where nginx runs |
| `nohup` | Keeps command running after logout |
| `port-forward.log` | Stores command output |

### Access Nginx

Open browser:

```
http://localhost:8080
```

Or test using curl:

```bash
curl http://localhost:8080
```

Expected output:

```
Welcome to nginx!
```

---

## Imperative vs Declarative Comparison

| Feature | Imperative | Declarative |
|---|---|---|
| Method | Command based | YAML based |
| Example | `kubectl run` | `kubectl apply` |
| Configuration | Stored in command history | Stored in files |
| Version Control | Difficult | Easy with Git |
| Reusability | Low | High |
| Production Usage | Mostly troubleshooting/testing | Preferred approach |

---

## Useful Kubernetes Commands

Check pods:
```bash
kubectl get pods
```

Detailed pod information:
```bash
kubectl describe pod <pod-name>
```

Delete pod:
```bash
kubectl delete pod <pod-name>
```

Apply YAML:
```bash
kubectl apply -f <file.yaml>
```

Delete using YAML:
```bash
kubectl delete -f <file.yaml>
```

---

## Summary

- **Imperative approach:** Tell Kubernetes exactly what to do using commands.
- **Declarative approach:** Define the required state in YAML and let Kubernetes manage the state.
- YAML-based declarative deployment is the **recommended approach for production** Kubernetes environments.
