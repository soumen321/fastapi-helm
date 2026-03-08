# End to End Workflow with GitOps CI/CD with Flux, Helm, Docker, and GitHub Actions

This project demonstrates a **complete GitOps-based CI/CD pipeline** for deploying a FastAPI application to Kubernetes using:

* **GitHub Actions** for CI/CD
* **Docker Hub** for container registry
* **Helm** for Kubernetes packaging
* **FluxCD** for GitOps-based deployment

The workflow automatically builds and pushes a Docker image, updates the Helm chart repository, and lets Flux reconcile the Kubernetes cluster.

---

# Architecture Overview

```
Developer Push Code
        │
        ▼
GitHub Actions CI Pipeline
        │
        ├── Build Docker Image
        ├── Push Image → Docker Hub
        ├── Update Helm values.yaml
        ├── Update Helm Chart.yaml
        │
        ▼
Helm Chart Repository Updated
        │
        ▼
FluxCD detects Git change
        │
        ▼
HelmRelease Reconciliation
        │
        ▼
Kubernetes Deploys New Pod
```

---

# Repository Structure

This setup uses **two repositories**.

## 1. Application Repository

Example: `fastapi-app`

```
fastapi-app
│
├── fastapi-app
│   ├── Dockerfile
│   ├── main.py
│   └── requirements.txt
│
└── .github
    └── workflows
        └── cicd.yaml
```

## 2. Helm Repository

Example: `fastapi-helm`

```
fastapi-helm
│
└── fastapi-chart
    │
    ├── Chart.yaml
    ├── values.yaml
    │
    └── templates
        ├── deployment.yaml
        ├── service.yaml
        └── ingress.yaml
```

---

# Docker Image

The Docker image is built from:

```
fastapi-app/Dockerfile
```

Example Dockerfile:

```
FROM python:3.10-slim

WORKDIR /app

COPY requirements.txt .

RUN pip install -r requirements.txt

COPY . .

CMD ["uvicorn","main:app","--host","0.0.0.0","--port","8000"]
```

---

# Helm Chart Configuration

## Chart.yaml

```
apiVersion: v2
name: fastapi-chart
description: FastAPI Helm Chart

version: 0.1.1
appVersion: "1.1"
```

## values.yaml

```
image:
  repository: soumen321/fastapi-demo
  tag: latest

service:
  type: ClusterIP
  port: 80
```

---

# FluxCD GitOps Setup

Flux watches the Helm repository and automatically reconciles when changes occur.

Flux resources typically include:

* GitRepository
* HelmRepository
* HelmRelease

Example HelmRelease:

```
apiVersion: helm.toolkit.fluxcd.io/v2
kind: HelmRelease
metadata:
  name: fastapi
  namespace: default

spec:
  interval: 1m
  chart:
    spec:
      chart: fastapi-chart
      sourceRef:
        kind: GitRepository
        name: fastapi-helm
```

Whenever the Helm repo changes, Flux redeploys the application.

---

# GitHub Actions CI/CD Pipeline

File:

```
.github/workflows/cicd.yaml
```

```
name: FastAPI CI/CD

on:
  push:
    branches:
      - main

  workflow_dispatch:

jobs:
  build-and-push:
    runs-on: ubuntu-latest

    steps:

    - name: Checkout Code
      uses: actions/checkout@v4

    - name: Login to DockerHub
      uses: docker/login-action@v3
      with:
        username: ${{ secrets.DOCKER_USERNAME }}
        password: ${{ secrets.DOCKER_PASSWORD }}

    - name: Build Docker Image
      run: |
        docker build -t ${{ secrets.DOCKER_USERNAME }}/fastapi-demo:${{ github.sha }} ./fastapi-app

    - name: Push Docker Image
      run: |
        docker push ${{ secrets.DOCKER_USERNAME }}/fastapi-demo:${{ github.sha }}

    - name: Clone Helm Repo
      run: |
        git clone https://${{ secrets.HELM_REPO_TOKEN }}@github.com/soumen321/fastapi-helm.git

    - name: Update Helm values.yaml
      run: |
        cd fastapi-helm/fastapi-chart
        sed -i "s/tag:.*/tag: ${{ github.sha }}/" values.yaml

    - name: Update Helm appVersion
      run: |
        cd fastapi-helm/fastapi-chart
        sed -i "s/appVersion:.*/appVersion: \"${{ github.sha }}\"/" Chart.yaml

    - name: Commit and Push Changes
      run: |
        cd fastapi-helm
        git config --global user.email "actions@github.com"
        git config --global user.name "github-actions"
        git add fastapi-chart/values.yaml fastapi-chart/Chart.yaml
        git commit -m "Update FastAPI image to ${{ github.sha }}" || echo "No changes"
        git push
```

---

# GitHub Secrets Required

Add these secrets in the repository settings.

```
Settings → Secrets → Actions
```

### Docker Hub Credentials

```
DOCKER_USERNAME
DOCKER_PASSWORD
```

### Helm Repository Access

Create a GitHub Personal Access Token and add:

```
HELM_REPO_TOKEN
```

---

# Deployment Flow

1. Developer pushes code
2. GitHub Actions starts CI pipeline
3. Docker image is built
4. Image is pushed to Docker Hub
5. Helm chart values.yaml updated
6. Helm repo commit pushed
7. Flux detects Git change
8. HelmRelease reconciles
9. Kubernetes deploys new pods

---

# Verify Deployment

Check pods:

```
kubectl get pods
```

Check image version:

```
kubectl describe pod <pod-name>
```

Expected output:

```
Image: soumen321/fastapi-demo:<commit-sha>
```

---

# Manual Pipeline Trigger

You can manually run the pipeline from:

```
GitHub → Actions → FastAPI CI/CD → Run workflow
```

---

# Advantages of This GitOps Setup

* Fully automated deployment
* Versioned infrastructure
* Easy rollback
* Declarative Kubernetes configuration
* CI/CD integrated with Git workflow

---

# Future Improvements

Recommended production enhancements:

* Flux Image Automation
* Helm chart version auto-bumping
* Multi-environment deployments (dev/stage/prod)
* GitHub OIDC authentication
* Container security scanning
* Helm chart testing

---

# Technologies Used

* GitHub Actions
* Docker
* Helm
* FluxCD
* Kubernetes
* FastAPI

---

# Author

Soumen Bhattacharjee
