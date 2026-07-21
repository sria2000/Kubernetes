# Kubernetes API Documentation & `kubectl explain`

## Official Kubernetes API Documentation

**Kubernetes API Reference (v1.32)**

https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.32/

The Kubernetes API documentation provides detailed information about every Kubernetes object, including:

- Pod
- Deployment
- Service
- ConfigMap
- Secret
- PersistentVolume
- StatefulSet
- DaemonSet
- Job
- CronJob
- Ingress
- RBAC Objects
- Storage Classes
- Networking APIs
- Custom Resources (CRDs)

---

# kubectl explain

`kubectl explain` is one of the most useful commands for Kubernetes administrators.

It displays the documentation directly from the Kubernetes API schema installed on your cluster.

Syntax:

```bash
kubectl explain <RESOURCE>
```

---

## Explain a Pod

```bash
kubectl explain pod
```

Example Output

```
KIND:     Pod
VERSION:  v1

DESCRIPTION:
    Pod is a collection of containers...
```

---

## Explain Pod Specification

```bash
kubectl explain pod.spec
```

Shows every field available inside the Pod specification, such as:

- containers
- volumes
- restartPolicy
- dnsPolicy
- nodeSelector
- affinity
- tolerations
- serviceAccountName
- securityContext

---

## Explain Volumes

```bash
kubectl explain pod.spec.volumes
```

This displays every supported volume type, including:

- emptyDir
- configMap
- secret
- persistentVolumeClaim
- hostPath
- projected
- nfs
- iscsi
- csi
- downwardAPI

---

## Explain Secret Volume

```bash
kubectl explain pod.spec.volumes.secret
```

Displays the fields available for a Secret volume.

Example:

- secretName
- optional
- items
- defaultMode

---

## Explain secretName

```bash
kubectl explain pod.spec.volumes.secret.secretName
```

Example Output

```
FIELD: secretName <string>

DESCRIPTION:
Name of the secret in the pod's namespace to use.
```

---

# Going Deeper

You can continue drilling into any field.

Examples:

```bash
kubectl explain deployment
```

```bash
kubectl explain deployment.spec
```

```bash
kubectl explain deployment.spec.template
```

```bash
kubectl explain deployment.spec.template.spec
```

```bash
kubectl explain deployment.spec.template.spec.containers
```

```bash
kubectl explain deployment.spec.template.spec.containers.resources
```

```bash
kubectl explain deployment.spec.template.spec.containers.resources.requests
```

---

# Recursive Output

To display all nested fields recursively:

```bash
kubectl explain pod --recursive
```

Example:

```bash
kubectl explain deployment.spec --recursive
```

This prints the complete hierarchy of fields.

---

# Practical Examples

View all fields under a Service:

```bash
kubectl explain service.spec
```

View Ingress rules:

```bash
kubectl explain ingress.spec.rules
```

View PersistentVolumeClaim:

```bash
kubectl explain pvc.spec
```

View ConfigMap:

```bash
kubectl explain configmap
```

View Secret:

```bash
kubectl explain secret
```

View Namespace:

```bash
kubectl explain namespace
```

---

# Why Use `kubectl explain`?

Benefits:

- No need to search the internet while working.
- Reads the schema directly from your cluster.
- Shows valid fields and their descriptions.
- Helps write YAML manifests correctly.
- Useful during Kubernetes exams (CKA/CKAD) and interviews.
- Great for learning unfamiliar Kubernetes resources.

---

# Summary

| Command | Purpose |
|----------|---------|
| `kubectl explain pod` | Display Pod documentation |
| `kubectl explain pod.spec` | Show Pod specification fields |
| `kubectl explain pod.spec.volumes` | List supported volume types |
| `kubectl explain pod.spec.volumes.secret` | Show Secret volume fields |
| `kubectl explain pod.spec.volumes.secret.secretName` | Explain the `secretName` field |
| `kubectl explain deployment.spec --recursive` | Display the full Deployment spec hierarchy |
| `kubectl explain pod --recursive` | Display the complete Pod schema |
| `kubectl explain service.spec` | Explain Service specification |
| `kubectl explain ingress.spec.rules` | Explain Ingress rules |
