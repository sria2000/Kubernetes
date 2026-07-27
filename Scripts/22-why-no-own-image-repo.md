# 12. Why does Kubernetes not have its own image repository and which registry is used by default?

**Why:** Kubernetes is a container *orchestrator*, not a container build/storage system. It was intentionally designed to be registry-agnostic — it just needs somewhere to pull OCI-compliant images from via the container runtime (containerd, CRI-O, etc.). Building and maintaining its own registry would duplicate work already done well by dedicated registry projects (Docker Hub, Harbor, Quay, Artifact Registry, ECR, GHCR), and would tie Kubernetes to one company's infrastructure — against its vendor-neutral, CNCF design philosophy.

**Default registry:** If no registry is specified in the image name, Kubernetes/containerd defaults to resolving it against **Docker Hub** (`docker.io`).

Example:
```
image: nginx:latest
```
is shorthand for:
```
image: docker.io/library/nginx:latest
```

To use a different registry, specify the full path, e.g. `gcr.io/...`, `<account>.dkr.ecr.<region>.amazonaws.com/...`, or a private registry — optionally with an `imagePullSecrets` entry if authentication is required.
