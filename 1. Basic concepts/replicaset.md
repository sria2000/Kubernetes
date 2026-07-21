# Kubernetes Labels, Selectors, and Naming Convention

One of the most confusing topics for Kubernetes beginners is understanding the different names used in a Deployment or ReplicaSet.

This guide explains each field using a real-world example.

---

# Real World Example

Imagine **TCS** is an IT company.

- **Deployment** = HR Manager
- **ReplicaSet** = Team Leader
- **Pods** = Employees
- **Container** = Laptop assigned to each employee
- **Image** = Standard Operating System (nginx)
- **Label** = Employee's Department

```
                    TCS Company
                         |
                  Deployment (HR)
                         |
                 ReplicaSet (Team Lead)
                         |
        +----------------+----------------+
        |                |                |
      Employee1       Employee2       Employee3
        (Pod)           (Pod)           (Pod)
           |               |               |
      employee-laptop employee-laptop employee-laptop
           |               |               |
          nginx           nginx           nginx
```

---

# Example Deployment YAML

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: tcs-deployment

spec:
  replicas: 3

  selector:
    matchLabels:
      department: web-team

  template:
    metadata:
      labels:
        department: web-team

    spec:
      containers:
      - name: employee-laptop
        image: nginx
        ports:
        - containerPort: 80
```

---

# Understanding Every Name

## 1. Deployment Name

```yaml
metadata:
  name: tcs-deployment
```

This is the name of the Deployment object.

View it using:

```bash
kubectl get deployments
```

Output:

```
NAME
tcs-deployment
```

---

## 2. ReplicaSet Name

You never create this manually.

The Deployment automatically creates it.

Example:

```
tcs-deployment-5776bb48d4
```

View it using:

```bash
kubectl get rs
```

Example:

```
NAME
tcs-deployment-5776bb48d4
```

---

## 3. Pod Names

Pods are automatically created by the ReplicaSet.

Example:

```
tcs-deployment-5776bb48d4-nmk8r
tcs-deployment-5776bb48d4-vbmgl
tcs-deployment-5776bb48d4-zttwb
```

View them:

```bash
kubectl get pods
```

Notice the naming convention:

```
Deployment Name
        |
        +-----------------------+
                                |
                       ReplicaSet Hash
                                |
                          Random Suffix
```

Example:

```
tcs-deployment-5776bb48d4-nmk8r
```

---

## 4. Container Name

Inside every Pod is one or more containers.

Container name:

```yaml
containers:
- name: employee-laptop
```

View it:

```bash
kubectl describe pod <pod-name>
```

Example:

```
Containers:

employee-laptop
```

This is **NOT** the Pod name.

---

## 5. Image

```yaml
image: nginx
```

This is the Docker image that Kubernetes downloads.

```
Docker Hub
     |
     |
   nginx
```

---

# What is a Label?

A label is simply a key-value pair attached to an object.

Example:

```yaml
labels:
  department: web-team
```

Think of it as an employee's department.

```
Employee

Name:
John

Department:
Web Team
```

Every Pod created by this Deployment gets this label.

---

# Selector

The Deployment says:

> "Manage every Pod whose department is web-team."

```yaml
selector:
  matchLabels:
    department: web-team
```

The Pods receive:

```yaml
labels:
  department: web-team
```

Because they match, the Deployment can manage them.

```
Deployment

Selector

department=web-team
        |
        |
        +----------------------+
        |          |           |
      Pod1       Pod2       Pod3
department=   department=  department=
web-team      web-team     web-team
```

---

# Viewing Labels

Show labels on every Pod

```bash
kubectl get pods --show-labels
```

Example:

```
NAME                                LABELS

tcs-deployment-5776bb48d4-nmk8r
department=web-team,pod-template-hash=5776bb48d4

tcs-deployment-5776bb48d4-vbmgl
department=web-team,pod-template-hash=5776bb48d4
```

---

# Find Pods by Label

List only Pods in the Web Team

```bash
kubectl get pods -l department=web-team
```

Output

```
tcs-deployment-5776bb48d4-nmk8r
tcs-deployment-5776bb48d4-vbmgl
tcs-deployment-5776bb48d4-zttwb
```

---

# Describe a Pod

```bash
kubectl describe pod tcs-deployment-5776bb48d4-nmk8r
```

Output

```
Labels:
    department=web-team
```

This confirms that the Pod has the label assigned by the Deployment.

---

# Relationship Between Components

```
Deployment
Name:
tcs-deployment

        |
        |
        v

ReplicaSet
Name:
tcs-deployment-5776bb48d4

        |
        |
        v

Pods

tcs-deployment-5776bb48d4-nmk8r

Label:
department=web-team

------------------------

tcs-deployment-5776bb48d4-vbmgl

Label:
department=web-team

------------------------

tcs-deployment-5776bb48d4-zttwb

Label:
department=web-team
```

---

# Summary

| Kubernetes Object | Example | Purpose |
|-------------------|---------|---------|
| Deployment | `tcs-deployment` | Manages application lifecycle |
| ReplicaSet | `tcs-deployment-5776bb48d4` | Maintains desired number of Pods |
| Pod | `tcs-deployment-5776bb48d4-nmk8r` | Runs the application |
| Container | `employee-laptop` | Container inside the Pod |
| Image | `nginx` | Docker image used to create the container |
| Label | `department=web-team` | Groups related Pods |
| Selector | `department=web-team` | Finds Pods with matching labels |

---

# Key Takeaway

- **Names** uniquely identify Kubernetes objects.
- **Labels** group related objects together.
- **Selectors** use labels to find and manage the correct Pods.
- **Images** define what software runs inside the container.
- **Containers** run inside Pods.
- **Deployments** create ReplicaSets.
- **ReplicaSets** create and maintain Pods.
