# Wisecow Application Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        GitHub Repository                        │
│  ┌─────────────┐  ┌──────────────┐  ┌─────────────────────────┐ │
│  │ wisecow.sh  │  │ Dockerfile   │  │ k8s-deployment.yaml     │ │
│  │ (App Code)  │  │ (Container)  │  │ (K8s Manifests)         │ │
│  └─────────────┘  └──────────────┘  └─────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
                                │
                                │ git push
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                      GitHub Actions CI/CD                      │
│                                                                 │
│  Step 1: Checkout Code                                          │
│  Step 2: Create Kind Cluster                                    │
│  Step 3: Build Docker Image                                     │
│  Step 4: Load Image to Kind                                     │
│  Step 5: Deploy to Kubernetes                                   │
└─────────────────────────────────────────────────────────────────┘
                                │
                                │ deploys to
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                       Kind Cluster                             │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                  Kubernetes                             │   │
│  │                                                         │   │
│  │  ┌─────────────────┐    ┌─────────────────────────┐    │   │
│  │  │   Deployment    │    │       Service           │    │   │
│  │  │                 │    │   (ClusterIP)           │    │   │
│  │  │  ┌───────────┐  │    │   Port: 80 → 4499       │    │   │
│  │  │  │    Pod    │  │    │                         │    │   │
│  │  │  │           │  │    └─────────────────────────┘    │   │
│  │  │  │ Container │  │                                   │   │
│  │  │  │ wisecow   │  │                                   │   │
│  │  │  │ :latest   │  │                                   │   │
│  │  │  │           │  │                                   │   │
│  │  │  │ Port:4499 │  │                                   │   │
│  │  │  └───────────┘  │                                   │   │
│  │  └─────────────────┘                                   │   │
│  └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘

Application Flow:
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   fortune   │───▶│   cowsay    │───▶│  netcat     │
│ (generates  │    │ (formats    │    │ (serves on  │
│  quotes)    │    │  output)    │    │  port 4499) │
└─────────────┘    └─────────────┘    └─────────────┘

Data Flow:
User Request ──▶ Service (Port 80) ──▶ Pod (Port 4499) ──▶ Wisecow App
                                                            │
                                                            ▼
User Response ◀── Service ◀────────────── Pod ◀─── Fortune + Cowsay

TLS Setup (Optional):
┌─────────────────────────────────────────────────────────────────┐
│                         Ingress                                │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────────────┐ │
│  │   TLS Cert  │    │   Ingress   │    │      Service        │ │
│  │  (tls.crt)  │───▶│ Controller  │───▶│   (wisecow-svc)     │ │
│  │  (tls.key)  │    │             │    │                     │ │
│  └─────────────┘    └─────────────┘    └─────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘

Network Architecture:
Internet ──▶ Ingress (HTTPS) ──▶ Service (HTTP) ──▶ Pod (App)
             │                    │                   │
             │ Port: 443          │ Port: 80          │ Port: 4499
             │ Protocol: HTTPS    │ Protocol: HTTP    │ Protocol: HTTP
```

## Component Details

### Application Layer
- **wisecow.sh**: Bash script serving fortune quotes with cowsay
- **Dependencies**: fortune-mod, cowsay, netcat
- **Port**: 4499

### Container Layer  
- **Base Image**: Ubuntu
- **Runtime**: Bash shell
- **Exposed Port**: 4499

### Kubernetes Layer
- **Deployment**: 1 replica, rolling updates
- **Service**: ClusterIP, port mapping 80→4499
- **Ingress**: TLS termination (optional)

### CI/CD Layer
- **Trigger**: Git push to main branch
- **Platform**: GitHub Actions
- **Cluster**: Kind (Kubernetes in Docker)
- **Strategy**: Build → Load → Deploy → Verify
