# DevSecOps GitOps Repository

A comprehensive Kubernetes-based DevSecOps infrastructure powered by ArgoCD with integrated tools for CI/CD, container registry, security scanning, and Kubernetes management.

## 📋 Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Tools Included](#tools-included)
- [Directory Structure](#directory-structure)
- [Prerequisites](#prerequisites)
- [Installation & Deployment](#installation--deployment)
- [Accessing Services](#accessing-services)
- [Configuration](#configuration)
- [Troubleshooting](#troubleshooting)

## 🎯 Overview

This repository contains Kubernetes manifests for deploying a complete DevSecOps stack on Kubernetes using ArgoCD for GitOps automation. Each tool is containerized and deployed as a Kubernetes service with automatic synchronization and self-healing capabilities.

## 🏗️ Architecture

```
Kubernetes Cluster
├── ArgoCD (GitOps Controller)
├── Jenkins (CI/CD Pipeline)
├── SonarQube (Code Quality)
├── Trivy (Container Security)
├── Harbor (Container Registry)
└── Headlamp (Kubernetes Dashboard)
```

## 🛠️ Tools Included

### 1. **Jenkins** (CI/CD Pipeline Automation)
- **Namespace**: jenkins
- **Service Type**: NodePort
- **Port**: 8080
- **NodePort**: 30081
- **Purpose**: Continuous Integration and Deployment automation
- **Features**: Pipeline as code, plugin ecosystem, distributed builds

### 2. **SonarQube** (Code Quality Analysis)
- **Namespace**: sonarqube
- **Service Type**: NodePort
- **Port**: 9000
- **NodePort**: 30083
- **Purpose**: Static code analysis and quality gate enforcement
- **Features**: Vulnerability detection, technical debt tracking, code metrics

### 3. **Trivy** (Container Security Scanning)
- **Namespace**: trivy
- **Service Type**: NodePort
- **Port**: 8080
- **NodePort**: 30084
- **Purpose**: Vulnerability scanning for container images and filesystems
- **Features**: Fast, comprehensive vulnerability database, SBOM generation

### 4. **Harbor** (Container Registry)
- **Namespace**: harbor
- **Service Type**: NodePort
- **Ports**: 80 (30080), 443 (30443)
- **Purpose**: Secure container image registry with policy enforcement
- **Features**: Image scanning, access control, replication, garbage collection

### 5. **Headlamp** (Kubernetes Dashboard)
- **Namespace**: headlamp
- **Service Type**: NodePort
- **Port**: 80
- **NodePort**: 30082
- **Purpose**: Modern web-based Kubernetes cluster management UI
- **Features**: Resource visualization, logs, terminal access, RBAC support

### 6. **ArgoCD** (GitOps Controller)
- **Namespace**: argocd
- **Purpose**: Continuous deployment and reconciliation from Git repositories
- **Features**: Automatic sync, health assessment, multi-cluster support

## 📁 Directory Structure

```
devsecops-gitops/
├── README.md
├── devsecops-tools/
│   ├── jenkins/
│   │   ├── service.yaml          # Jenkins Service definition
│   │   ├── jenkins-app.yaml      # ArgoCD Application manifest
│   │   ├── deployment.yaml       # Jenkins Deployment
│   │   └── namespace.yaml        # Jenkins namespace
│   ├── sonarqube/
│   │   ├── service.yaml          # SonarQube Service
│   │   └── sonarqube-app.yaml    # ArgoCD Application
│   ├── trivy/
│   │   ├── service.yaml          # Trivy Service
│   │   └── trivy-app.yaml        # ArgoCD Application
│   ├── harbor/
│   │   ├── service.yaml          # Harbor Service
│   │   └── harbor-app.yaml       # ArgoCD Application
│   └── headlamp/
│       ├── service.yaml          # Headlamp Service
│       └── headlamp-app.yaml     # ArgoCD Application
└── argocd-apps/
    └── (ArgoCD application configurations)
```

## 📋 Prerequisites

- **Kubernetes Cluster** (v1.20+)
  - Minikube, EKS, GKE, or any production Kubernetes cluster
- **kubectl** configured with cluster access
- **ArgoCD** installed on your cluster
- **Git** version control setup
- **Docker** (for building custom images)
- **Helm** (optional, for package management)

## 🚀 Installation & Deployment

### 1. Prerequisites Setup

```bash
# Install ArgoCD (if not already installed)
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Wait for ArgoCD to be ready
kubectl wait --for=condition=ready pod -l app.kubernetes.io/name=argocd-server -n argocd --timeout=300s
```

### 2. Clone the Repository

```bash
git clone https://github.com/Skpniet/devsecops-gitops.git
cd devsecops-gitops
```

### 3. Deploy Individual Applications

#### Option A: Using kubectl directly

```bash
# Apply all Jenkins manifests
kubectl apply -f devsecops-tools/jenkins/

# Apply other tools
kubectl apply -f devsecops-tools/sonarqube/
kubectl apply -f devsecops-tools/trivy/
kubectl apply -f devsecops-tools/harbor/
kubectl apply -f devsecops-tools/headlamp/
```

#### Option B: Using ArgoCD (Recommended)

```bash
# Apply ArgoCD Application manifests
kubectl apply -f devsecops-tools/jenkins/jenkins-app.yaml
kubectl apply -f devsecops-tools/sonarqube/sonarqube-app.yaml
kubectl apply -f devsecops-tools/trivy/trivy-app.yaml
kubectl apply -f devsecops-tools/harbor/harbor-app.yaml
kubectl apply -f devsecops-tools/headlamp/headlamp-app.yaml
```

### 4. Verify Deployments

```bash
# Check all deployments
kubectl get deployments -A

# Monitor ArgoCD applications
kubectl get applications -n argocd

# View pods across all namespaces
kubectl get pods -A
```

## 🌐 Accessing Services

### Local Access (Minikube/Development)

```bash
# Get Minikube IP
minikube ip

# Access services using NodePort:
# Jenkins:    http://<MINIKUBE_IP>:30081
# SonarQube:  http://<MINIKUBE_IP>:30083
# Trivy:      http://<MINIKUBE_IP>:30084
# Harbor:     http://<MINIKUBE_IP>:30080
# Headlamp:   http://<MINIKUBE_IP>:30082
```

### Production Access (Cloud Clusters)

```bash
# Get worker node IP
kubectl get nodes -o wide

# Use worker node IP with NodePort
# Example: http://<WORKER_NODE_IP>:30081
```

### Port Forwarding (Alternative)

```bash
# Jenkins
kubectl port-forward -n jenkins svc/jenkins-service 8080:8080

# SonarQube
kubectl port-forward -n sonarqube svc/sonarqube-service 9000:9000

# Harbor
kubectl port-forward -n harbor svc/harbor-service 80:80

# Headlamp
kubectl port-forward -n headlamp svc/headlamp-service 8082:80

# Then access via http://localhost:PORT
```

## ⚙️ Configuration

### Customizing Deployments

Edit the respective `deployment.yaml` or `service.yaml` files in each tool directory:

```bash
# Example: Customize Jenkins deployment
kubectl edit deployment jenkins -n jenkins

# Or edit the file and reapply
vim devsecops-tools/jenkins/deployment.yaml
kubectl apply -f devsecops-tools/jenkins/
```

### ArgoCD Sync Configuration

All applications use automated sync with these policies:

```yaml
syncPolicy:
  automated:
    prune: true       # Delete resources not in Git
    selfHeal: true    # Auto-sync when cluster differs from Git
```

To modify sync policies, edit the respective `*-app.yaml` files.

### Updating GitHub Repository

```bash
# Make changes to manifests
git add .
git commit -m "Update: [description of changes]"
git push origin main

# ArgoCD will automatically sync (if automated sync is enabled)
```

## 🔐 Security Best Practices

1. **Image Registry Security**
   - Enable Harbor's vulnerability scanning
   - Use private registry credentials
   - Implement image pull secrets

2. **Code Quality Gates**
   - Configure SonarQube quality gates in Jenkins
   - Enforce minimum code coverage
   - Block deployment on critical issues

3. **Container Scanning**
   - Scan all images with Trivy before deployment
   - Block images with high/critical vulnerabilities
   - Maintain SBOM for compliance

4. **Access Control**
   - Use RBAC in Kubernetes and ArgoCD
   - Implement network policies
   - Secure Jenkins with authentication

5. **Secrets Management**
   - Use Kubernetes Secrets or external secret managers
   - Never commit secrets to Git
   - Rotate credentials regularly

## 📊 Service Ports Reference

| Tool | Namespace | Internal Port | NodePort | Protocol |
|------|-----------|---------------|----------|----------|
| Jenkins | jenkins | 8080 | 30081 | HTTP |
| SonarQube | sonarqube | 9000 | 30083 | HTTP |
| Trivy | trivy | 8080 | 30084 | HTTP |
| Harbor | harbor | 80/443 | 30080/30443 | HTTP/HTTPS |
| Headlamp | headlamp | 80 | 30082 | HTTP |
| ArgoCD | argocd | 443 | - | HTTPS |

## 🐛 Troubleshooting

### Pod Not Starting

```bash
# Check pod status
kubectl describe pod <POD_NAME> -n <NAMESPACE>

# View pod logs
kubectl logs <POD_NAME> -n <NAMESPACE>

# Check events
kubectl get events -n <NAMESPACE> --sort-by='.lastTimestamp'
```

### Service Not Accessible

```bash
# Verify service is created
kubectl get svc -n <NAMESPACE>

# Check endpoint connectivity
kubectl get endpoints -n <NAMESPACE>

# Test service connectivity
kubectl run -it --rm debug --image=busybox --restart=Never -- wget -O- http://<SERVICE_NAME>.<NAMESPACE>.svc.cluster.local:PORT
```

### ArgoCD Application Out of Sync

```bash
# Check application status
kubectl get applications -n argocd

# View detailed application info
kubectl describe application <APP_NAME> -n argocd

# Manual sync
argocd app sync <APP_NAME>
```

### Persistent Storage Issues

```bash
# Check PVC status
kubectl get pvc -A

# Check PV status
kubectl get pv

# Describe PVC for details
kubectl describe pvc <PVC_NAME> -n <NAMESPACE>
```

## 📚 Resources & Documentation

- [Kubernetes Documentation](https://kubernetes.io/docs/)
- [ArgoCD Documentation](https://argo-cd.readthedocs.io/)
- [Jenkins Documentation](https://www.jenkins.io/doc/)
- [SonarQube Documentation](https://docs.sonarqube.org/)
- [Harbor Documentation](https://goharbor.io/docs/)
- [Trivy Documentation](https://aquasecurity.github.io/trivy/)
- [Headlamp Documentation](https://headlamp.dev/)

## 🤝 Contributing

1. Create a feature branch: `git checkout -b feature/improvement`
2. Make your changes and test
3. Commit with descriptive message: `git commit -m 'Add: improvement'`
4. Push to branch: `git push origin feature/improvement`
5. Submit a Pull Request

## 📝 License

This project is licensed under the MIT License - see LICENSE file for details.

## 📞 Support & Contact

For issues, questions, or contributions:
- GitHub Issues: [devsecops-gitops Issues](https://github.com/Skpniet/devsecops-gitops/issues)
- Email: [your-email@example.com]

---

**Last Updated**: January 29, 2026
**Version**: 1.0.0
**Maintainer**: Skpniet
