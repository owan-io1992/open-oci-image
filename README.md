# argocd integrate helm-secret

## Overview

This Docker image extends the official `argocd-repo-server` to provide native support for decrypting Helm secrets using `sops`. It allows you to store your sensitive values encrypted in a Git repository and have Argo CD decrypt them on the fly during manifest generation.

This image is automatically built and published to Docker Hub.

## How it Works

The integration is achieved by installing `sops` and the `helm-secrets` plugin into the Argo CD repo server container. A wrapper script intercepts Helm commands, allowing `helm-secrets` to automatically decrypt any files named `secrets.yaml` or matching configured patterns before Helm renders the chart.

### Pre-installed Tools
- **sops**: For encrypting and decrypting files. (Source: https://github.com/getsops/sops)
- **helm-secrets**: The Helm plugin that enables secret decryption. (Source: https://github.com/jkroepke/helm-secrets)
- **kubectl**: The Kubernetes CLI.

The versions of these tools are defined in the `argocd-repo-server.docker-compose.yaml` file.

## Usage

### 1. Update Argo CD Repo Server Image
Replace the standard `argocd-repo-server` image in your Argo CD deployment with this custom image.

```yaml
# In your Argo CD deployment manifest
spec:
  template:
    spec:
      containers:
      - name: argocd-repo-server
        image: owanio1992/argocd-repo-server:v2.11.2 # Replace with your username and desired version
```

### 2. Configure Argo CD Application
To instruct Helm to decrypt a file, you must use the `secrets://` prefix for the file path in your `values.yaml` file or directly in the Argo CD Application manifest's `helm.valueFiles` section.

**Example `values.yaml`:**
```yaml
# values.yaml
some_regular_value: "hello"

# The helm-secrets plugin will decrypt this file
encrypted_values: "secrets://path/to/your/secrets.yaml"
```

**Example Argo CD Application:**
```yaml
# app.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: 'https://your-git-repo.git'
    targetRevision: HEAD
    path: helm-chart
    helm:
      valueFiles:
        - values.yaml
        - secrets://path/to/your/secrets.yaml
  destination:
    server: 'https://kubernetes.default.svc'
    namespace: my-app-namespace
```

## Build Process

The image is built weekly via a GitHub Actions workflow. The workflow checks for the latest Argo CD release. If a new release is found, it builds and pushes a new image to Docker Hub, tagged with the corresponding Argo CD version.

## Read More

For more detailed information on integrating `helm-secrets` with Argo CD, please refer to the official documentation:

- [helm-secrets Wiki: ArgoCD Integration](https://github.com/jkroepke/helm-secrets/wiki/ArgoCD-Integration)