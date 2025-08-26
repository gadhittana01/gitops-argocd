# GitOps ArgoCD Repository

This repository contains the ArgoCD configuration and Kubernetes manifests for deploying the GitOps practical assignment applications.

## Repository Structure

```
gitops-argocd/
├── argocd/
│   ├── applications/
│   │   ├── simple-nginx-app.yaml
│   │   └── simple-httpd-app.yaml
│   └── projects/
│       └── gitops-project.yaml
├── k8s/
│   ├── namespace.yaml
│   ├── simple-nginx/
│   │   ├── deployment.yaml
│   │   └── service.yaml
│   └── simple-httpd/
│       ├── deployment.yaml
│       └── service.yaml
└── README.md
```

## Architecture

### Multi-Repository GitOps Setup

This repository implements a **multi-repository GitOps pattern**:

1. **Applications Repository** (`gitops-applications`): Contains application source code and CI/CD
2. **GitOps Repository** (`gitops-argocd`): Contains ArgoCD configuration and K8s manifests

### Repository Separation Benefits

- **Separation of Concerns**: Application code vs. deployment configuration
- **Security**: Different access controls for developers vs. operators
- **Scalability**: Independent versioning and release cycles
- **Compliance**: Clear audit trail for infrastructure changes

## ArgoCD Configuration

### Project (`argocd/projects/gitops-project.yaml`)
- **Name**: `gitops-project`
- **Namespace**: `argocd`
- **Source Repositories**: 
  - `gitops-applications` (for application images)
  - `gitops-argocd` (for deployment manifests)
- **Destination**: `gitops-project` namespace in the cluster

### Applications

#### Simple NGINX (`argocd/applications/simple-nginx-app.yaml`)
- **Source**: This repository (`gitops-argocd`)
- **Path**: `k8s/simple-nginx`
- **Image**: `ghcr.io/your-username/gitops-applications/simple-nginx:main`
- **Sync Policy**: Automated with pruning and self-healing

#### Simple HTTPD (`argocd/applications/simple-httpd-app.yaml`)
- **Source**: This repository (`gitops-argocd`)
- **Path**: `k8s/simple-httpd`
- **Image**: `ghcr.io/your-username/gitops-applications/simple-httpd:main`
- **Sync Policy**: Automated with pruning and self-healing

## Kubernetes Manifests

### Namespace (`k8s/namespace.yaml`)
- **Name**: `gitops-project`
- **Purpose**: Resource isolation and organization

### Deployments
Both applications use:
- **Replicas**: 2 (high availability)
- **Health Checks**: Liveness and readiness probes
- **Resource Limits**: CPU and memory constraints
- **Custom Images**: From the applications repository

### Services
- **Type**: NodePort
- **NGINX**: Port 30080
- **HTTPD**: Port 30081

## Deployment Process

### 1. Application Development
```bash
# In gitops-applications repository
git add .
git commit -m "Update application code"
git push origin main
```

### 2. CI/CD Pipeline
- GitHub Actions builds Docker images
- Images pushed to GHCR with version tags
- ArgoCD webhook triggered (optional)

### 3. ArgoCD Sync
- ArgoCD detects changes in this repository
- Validates Kubernetes manifests
- Applies changes to cluster
- Reports deployment status

### 4. Verification
- Applications accessible on NodePorts
- Health checks confirm successful deployment
- Monitoring shows application status

## Setup Instructions

### Prerequisites
- Kubernetes cluster with ArgoCD installed
- Access to GitHub Container Registry
- kubectl configured

### 1. Deploy ArgoCD Project
```bash
kubectl apply -f argocd/projects/gitops-project.yaml
```

### 2. Deploy Applications
```bash
kubectl apply -f argocd/applications/simple-nginx-app.yaml
kubectl apply -f argocd/applications/simple-httpd-app.yaml
```

### 3. Verify Deployment
```bash
# Check ArgoCD applications
kubectl get applications -n argocd

# Check application pods
kubectl get pods -n gitops-project

# Check services
kubectl get services -n gitops-project
```

## Access Applications

- **NGINX**: `http://<cluster-ip>:30080`
- **HTTPD**: `http://<cluster-ip>:30081`

## Monitoring and Observability

### ArgoCD Dashboard
- Application health status
- Sync status and history
- Resource health indicators
- Deployment logs

### Kubernetes Resources
- Pod status and logs
- Service endpoints
- Resource utilization
- Events and alerts

## Best Practices

### 1. Repository Management
- Use semantic versioning for releases
- Tag important releases
- Maintain clean commit history
- Use pull requests for changes

### 2. Security
- Limit repository access
- Use RBAC for ArgoCD
- Scan images for vulnerabilities
- Monitor resource usage

### 3. Deployment Strategy
- Use rolling updates
- Implement health checks
- Set resource limits
- Monitor application metrics

## Troubleshooting

### Common Issues

#### 1. Image Pull Errors
```bash
# Check image availability
docker pull ghcr.io/your-username/gitops-applications/simple-nginx:main

# Check pod events
kubectl describe pod -n gitops-project <pod-name>
```

#### 2. ArgoCD Sync Issues
```bash
# Check application status
kubectl get applications -n argocd

# View application logs
kubectl logs -n argocd deployment/argocd-application-controller
```

#### 3. Service Access Issues
```bash
# Check service endpoints
kubectl get endpoints -n gitops-project

# Test service connectivity
kubectl port-forward -n gitops-project svc/simple-nginx-service 8080:80
```

## Automation Scripts

### Deployment Script
```bash
#!/bin/bash
# deploy.sh
kubectl apply -f argocd/projects/gitops-project.yaml
kubectl apply -f argocd/applications/
```

### Cleanup Script
```bash
#!/bin/bash
# cleanup.sh
kubectl delete -f argocd/applications/
kubectl delete -f argocd/projects/
kubectl delete namespace gitops-project
```

## Future Enhancements

1. **Ingress Configuration**: Add ingress for external access
2. **Monitoring Stack**: Integrate Prometheus and Grafana
3. **Secrets Management**: Use external secrets operator
4. **Multi-Environment**: Support for staging and production
5. **Backup Strategy**: Implement disaster recovery procedures
