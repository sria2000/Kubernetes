# Helm Chart → GitHub Container Registry (GHCR)

## Overview

This guide shows how to:

1. Create a GitHub Personal Access Token (PAT)
2. Create a Helm chart
3. Package the Helm chart
4. Login to GitHub Container Registry (`ghcr.io`)
5. Push the Helm chart to GHCR
6. Pull the Helm chart from GHCR

The architecture is:

```text
Local Kubernetes Machine
        |
        | helm package
        v
mychart-1.1.0.tgz
        |
        | helm push
        v
GitHub Container Registry
        |
        v
ghcr.io/sria2000/charts/mychart:1.1.0
        |
        | helm pull
        v
Local Machine
```

---

# 1. GitHub Personal Access Token

Go to:

```text
GitHub
  → Settings
  → Developer Settings
  → Personal Access Tokens
```

Create a token with the required permissions.

For a classic Personal Access Token, use:

```text
write:packages
read:packages
```

Optional:

```text
delete:packages
```

### Permissions

| Permission        | Purpose                        |
| ----------------- | ------------------------------ |
| `write:packages`  | Push packages/images to GHCR   |
| `read:packages`   | Pull packages/images from GHCR |
| `delete:packages` | Delete packages from GHCR      |

---

# 2. Keep the Token Secure

Do **NOT** put your real token into a Git repository or Markdown file.

For example:

```text
<TOKEN>
```

should represent your real GitHub token.

Never commit something like:

```text
ghp_xxxxxxxxxxxxxxxxxxxxxxxxx
```

to GitHub.

If a token is accidentally exposed, revoke it immediately and create a new one.

---

# 3. Create a Helm Chart

Create a new Helm chart:

```bash
helm create mychart
```

This creates:

```text
mychart/
├── Chart.yaml
├── values.yaml
├── charts/
├── templates/
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   ├── serviceaccount.yaml
│   ├── hpa.yaml
│   ├── NOTES.txt
│   └── _helpers.tpl
└── .helmignore
```

Check:

```bash
ls -l mychart
```

---

# 4. Check Chart.yaml

Go into the chart:

```bash
cd mychart
```

View the chart:

```bash
cat Chart.yaml
```

Example:

```yaml
apiVersion: v2
name: mychart
description: A Helm chart for Kubernetes

type: application

version: 1.1.0

appVersion: "1.0.0"
```

The important field for this example is:

```yaml
version: 1.1.0
```

---

# 5. Package the Helm Chart

Go back to the directory containing `mychart`:

```bash
cd ..
```

Package the chart:

```bash
helm package mychart
```

Expected output:

```text
Successfully packaged chart and saved it to:
/root/mychart-1.1.0.tgz
```

Check:

```bash
ls -l
```

You should see:

```text
mychart/
mychart-1.1.0.tgz
```

The `.tgz` file is the packaged Helm chart.

---

# 6. Login to GitHub Container Registry

GitHub Container Registry uses:

```text
ghcr.io
```

Login:

```bash
helm registry login ghcr.io
```

You will be prompted:

```text
Username:
Password:
```

Enter your GitHub username:

```text
sria2000
```

For the password, enter your **GitHub Personal Access Token**.

You should see:

```text
Login Succeeded
```

---

# 7. Push the Helm Chart to GHCR

The registry path will be:

```text
ghcr.io/sria2000/charts
```

Push the packaged chart:

```bash
helm push mychart-1.1.0.tgz \
  oci://ghcr.io/sria2000/charts
```

Expected output:

```text
Pushed: ghcr.io/sria2000/charts/mychart:1.1.0
Digest: sha256:365bc1df4b...
```

The chart is now stored in GitHub Container Registry.

The full OCI reference is:

```text
oci://ghcr.io/sria2000/charts/mychart:1.1.0
```

---

# 8. Understand the GHCR Path

The command:

```bash
helm push mychart-1.1.0.tgz \
  oci://ghcr.io/sria2000/charts
```

results in:

```text
ghcr.io
   |
   +-- sria2000
          |
          +-- charts
                 |
                 +-- mychart
                        |
                        +-- 1.1.0
```

Therefore:

```text
Registry:
ghcr.io

GitHub user:
sria2000

Repository/path:
charts

Chart:
mychart

Version:
1.1.0
```

Final reference:

```text
ghcr.io/sria2000/charts/mychart:1.1.0
```

---

# 9. Pull the Helm Chart

Once the chart has been pushed, another machine can pull it.

First login:

```bash
helm registry login ghcr.io
```

Then:

```bash
helm pull \
  oci://ghcr.io/sria2000/charts/mychart \
  --version 1.1.0
```

Expected:

```text
Pulled: ghcr.io/sria2000/charts/mychart:1.1.0
Digest: sha256:365bc1df4b...
```

You should now have:

```text
mychart-1.1.0.tgz
```

---

# 10. Extract the Pulled Chart

You can extract the chart:

```bash
tar -xzf mychart-1.1.0.tgz
```

This creates:

```text
mychart/
```

Check:

```bash
ls -l mychart
```

---

# 11. Pull and Extract in One Command

Instead of downloading only the `.tgz`, you can use:

```bash
helm pull \
  oci://ghcr.io/sria2000/charts/mychart \
  --version 1.1.0 \
  --untar
```

This gives:

```text
mychart/
```

directly.

---

# 12. Verify the Chart

After pulling:

```bash
helm lint mychart
```

Expected:

```text
1 chart(s) linted, 0 chart(s) failed
```

You can also inspect it:

```bash
helm show chart mychart
```

Example:

```text
apiVersion: v2
name: mychart
version: 1.1.0
appVersion: 1.0.0
```

---

# 13. Install the Chart Directly from GHCR

You don't necessarily need to pull the chart first.

You can install directly from GHCR:

```bash
helm install my-release \
  oci://ghcr.io/sria2000/charts/mychart \
  --version 1.1.0
```

Check:

```bash
helm list
```

Check Kubernetes resources:

```bash
kubectl get pods
```

```bash
kubectl get deployments
```

```bash
kubectl get services
```

---

# 14. Upgrade the Chart

Suppose you change the chart and increase the version:

```yaml
version: 1.2.0
```

Package:

```bash
helm package mychart
```

This creates:

```text
mychart-1.2.0.tgz
```

Push:

```bash
helm push mychart-1.2.0.tgz \
  oci://ghcr.io/sria2000/charts
```

Then upgrade:

```bash
helm upgrade my-release \
  oci://ghcr.io/sria2000/charts/mychart \
  --version 1.2.0
```

---

# 15. Complete Workflow

The entire process is:

```text
                    GitHub
                      |
                      |
             Personal Access Token
                      |
                      v
                GHCR: ghcr.io
                      |
                      |
Local Machine         |
     |                |
     | helm create    |
     v                |
  mychart/            |
     |                |
     | helm package   |
     v                |
mychart-1.1.0.tgz     |
     |                |
     | helm push      |
     +--------------->|
                      |
                      v
        ghcr.io/sria2000/charts/mychart
                      |
                      |
                      | helm pull
                      v
               Another Machine
```

---

# 16. Complete Command List

## Create Chart

```bash
helm create mychart
```

## Enter Chart

```bash
cd mychart
```

## Change Version

Edit:

```bash
vi Chart.yaml
```

Set:

```yaml
version: 1.1.0
```

## Package

From the parent directory:

```bash
cd ..
```

```bash
helm package mychart
```

Result:

```text
mychart-1.1.0.tgz
```

## Login

```bash
helm registry login ghcr.io
```

Username:

```text
sria2000
```

Password:

```text
<TOKEN>
```

## Push

```bash
helm push mychart-1.1.0.tgz \
  oci://ghcr.io/sria2000/charts
```

Result:

```text
ghcr.io/sria2000/charts/mychart:1.1.0
```

## Pull

```bash
helm pull \
  oci://ghcr.io/sria2000/charts/mychart \
  --version 1.1.0
```

## Pull and Extract

```bash
helm pull \
  oci://ghcr.io/sria2000/charts/mychart \
  --version 1.1.0 \
  --untar
```

## Install Directly

```bash
helm install my-release \
  oci://ghcr.io/sria2000/charts/mychart \
  --version 1.1.0
```

---

# 17. Useful Helm OCI Commands

Check Helm version:

```bash
helm version
```

Login:

```bash
helm registry login ghcr.io
```

Logout:

```bash
helm registry logout ghcr.io
```

Package:

```bash
helm package mychart
```

Push:

```bash
helm push mychart-1.1.0.tgz \
  oci://ghcr.io/sria2000/charts
```

Pull:

```bash
helm pull \
  oci://ghcr.io/sria2000/charts/mychart \
  --version 1.1.0
```

Install:

```bash
helm install my-release \
  oci://ghcr.io/sria2000/charts/mychart \
  --version 1.1.0
```

Upgrade:

```bash
helm upgrade my-release \
  oci://ghcr.io/sria2000/charts/mychart \
  --version 1.2.0
```

---

# 18. Security Notes

## Never commit the token

Do not put this in:

```text
README.md
values.yaml
Chart.yaml
Git repository
Shell scripts
```

For example, avoid:

```bash
helm registry login ghcr.io \
  --username sria2000 \
  --password ghp_xxxxxxxxxxxxx
```

because the token can end up in shell history or logs.

---

## Use an environment variable when scripting

For automation, use:

```bash
export GHCR_TOKEN="<TOKEN>"
```

Then use the token through your CI/CD secret mechanism rather than hard-coding it.

For GitHub Actions, store the token as a repository/environment secret rather than putting it directly in the workflow.

---

# 19. Important Versioning Concept

The Helm chart version comes from:

```yaml
version: 1.1.0
```

When you run:

```bash
helm package mychart
```

Helm creates:

```text
mychart-1.1.0.tgz
```

When you push it:

```bash
helm push mychart-1.1.0.tgz \
  oci://ghcr.io/sria2000/charts
```

GHCR stores:

```text
mychart:1.1.0
```

Therefore:

```text
Chart.yaml
     |
     | version: 1.1.0
     v
mychart-1.1.0.tgz
     |
     | helm push
     v
ghcr.io/sria2000/charts/mychart:1.1.0
```

---

# 20. Final Architecture

```text
                    GITHUB
                      |
                      v
             Personal Access Token
                      |
                      v
              GitHub Container
                 Registry
                  GHCR
              ghcr.io/sria2000
                      |
                      v
                   charts
                      |
                      v
                  mychart
                      |
             +--------+--------+
             |                 |
           1.1.0             1.2.0
             |                 |
             v                 v
       Helm Chart          Helm Chart
```

The main commands to remember are:

```bash
helm create mychart
```

```bash
helm package mychart
```

```bash
helm registry login ghcr.io
```

```bash
helm push mychart-1.1.0.tgz \
  oci://ghcr.io/sria2000/charts
```

```bash
helm pull \
  oci://ghcr.io/sria2000/charts/mychart \
  --version 1.1.0
```

And the final OCI chart location is:

```text
oci://ghcr.io/sria2000/charts/mychart:1.1.0
```
