# FastAPI Deployment on Kubernetes using Helm

This project demonstrates how to deploy a **FastAPI application to Kubernetes using Helm**.
The workflow includes building a Docker image, pushing it to DockerHub, creating a Helm chart, and deploying it into a Kubernetes cluster.

---

# Architecture Overview

Developer → Docker Build → DockerHub → Helm Chart → Kubernetes Deployment

---

# 1. Prerequisites

Install the following tools:

* Docker
* Kubernetes cluster (Kind / Minikube / EKS)
* kubectl
* Helm
* Git

Verify installation:

```bash
kubectl version --client
helm version
docker version
```

---

# 2. FastAPI Application

Example project structure:

```
fastapi-app
 ├── app
 │   └── main.py
 ├── requirements.txt
 └── Dockerfile
```

Example `main.py`

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
def read_root():
    return {"message": "Hello from FastAPI Helm Deployment"}
```

---

# 3. Dockerfile

```
FROM python:3.10

WORKDIR /app

COPY requirements.txt .
RUN pip install -r requirements.txt

COPY app /app

CMD ["uvicorn","main:app","--host","0.0.0.0","--port","8000"]
```

---

# 4. Build Docker Image

```
docker build -t <dockerhub-username>/fastapi-demo:v1 .
```

Example

```
docker build -t soumen/fastapi-demo:v1 .
```

---

# 5. Push Image to DockerHub

Login:

```
docker login
```

Push image:

```
docker push <>/fastapi-demo:v1
```

---

# 6. Create Helm Chart

Generate Helm chart skeleton:

```
helm create fastapi-chart
```

Directory structure created:

```
fastapi-chart
├── Chart.yaml
├── values.yaml
├── templates
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   ├── hpa.yaml
│   └── _helpers.tpl
```

---

# 7. Modify Chart.yaml

```
apiVersion: v2
name: fastapi-chart
description: Helm chart for FastAPI application
type: application
version: 0.1.0
appVersion: "1.0"
```

---

# 8. Update values.yaml

```
replicaCount: 2

image:
  repository: <>/fastapi-demo
  tag: v1
  pullPolicy: IfNotPresent

service:
  type: NodePort
  port: 80
  targetPort: 8000
```

---

# 9. Configure Deployment Template

File:

```
templates/deployment.yaml
```

Important section:

```
containers:
  - name: fastapi
    image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
    ports:
      - containerPort: 8000
```

---

# 10. Configure Service Template

File:

```
templates/service.yaml
```

Example:

```
apiVersion: v1
kind: Service

metadata:
  name: fastapi-service

spec:
  type: NodePort

  selector:
    app: fastapi

  ports:
    - port: 80
      targetPort: 8000
      nodePort: 30007
```

---

# 11. Lint Helm Chart

Validate chart:

```
helm lint fastapi-chart
```

Expected result:

```
1 chart(s) linted, 0 chart(s) failed
```

---

# 12. Render Kubernetes Templates

Generate Kubernetes YAML without deploying:

```
helm template fastapi ./fastapi-chart
```

---

# 13. Install Helm Chart

Deploy application to Kubernetes:

```
helm install fastapi ./fastapi-chart
```

---

# 14. Verify Deployment

Check pods:

```
kubectl get pods
```

Check services:

```
kubectl get svc
```

---

# 15. Access Application

Find NodePort:

```
kubectl get svc
```

Example output:

```
fastapi-service   NodePort   80:30007
```

Access application:

```
http://localhost:30007
```

---

# 16. Upgrade Application

Update image version in `values.yaml`:

```
tag: v2
```

Upgrade deployment:

```
helm upgrade fastapi ./fastapi-chart
```

---

# 17. Uninstall Deployment

Remove Helm release:

```
helm uninstall fastapi
```

---

# Repository Structure

```
fastapi-helm
├── fastapi-chart
│   ├── Chart.yaml
│   ├── values.yaml
│   └── templates
│        ├── deployment.yaml
│        └── service.yaml
└── README.md
```

---

# Future Improvements

* Add Ingress controller
* Add Horizontal Pod Autoscaler
* Implement GitOps using FluxCD
* Integrate CI/CD pipeline
* Add monitoring with Prometheus and Grafana

---

# Author

Soumen Bhattacharjee
DevOps & Cloud Consultant
