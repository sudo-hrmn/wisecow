# Wisecow Application - Kubernetes Deployment

## Architecture Diagram
![Architecture Diagram](architecture%20diagram.png)

## Overview
Containerized deployment of the Wisecow application on Kubernetes with CI/CD pipeline and TLS support.

## Prerequisites
```bash
sudo apt install fortune-mod cowsay -y
```

## Local Testing
```bash
# Run locally
./wisecow.sh

# Build Docker image
docker build -t wisecow:latest .

# Run container
docker run -p 4499:4499 wisecow:latest
```

## Kubernetes Deployment
```bash
# Deploy to Kubernetes
kubectl apply -f k8s-deployment.yaml

# Enable TLS (optional)
kubectl create secret tls wisecow-tls --cert=tls.crt --key=tls.key
kubectl apply -f k8s-ingress.yaml
```

## CI/CD Pipeline
GitHub Actions automatically builds and deploys on push to main branch using Kind cluster.

## Files
- `Dockerfile` - Container image
- `k8s-deployment.yaml` - Kubernetes deployment and service
- `k8s-ingress.yaml` - TLS ingress configuration
- `.github/workflows/wisecow-deploy.yml` - CI/CD pipeline
- `ARCHITECTURE.md` - Detailed architecture documentation
