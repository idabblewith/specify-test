# Herbarium Specify Kubernetes Configuration

Kubernetes deployment configuration for the Herbarium Specify application stack using Kustomize.

## Overview

This repository contains the Kubernetes manifests and Kustomize overlays for deploying the Specify application stack, which includes:

- **Specify7 Web Application** - Main Django web application
- **Specify Worker** - Celery background task worker
- **Redis** - Message broker and cache
- **Nginx** - Reverse proxy and static file server
- **Asset Server** - File attachment management
- **Report Runner** - Report generation service
- **Specify6** - Legacy compatibility service

## Architecture

The application follows a microservices architecture with:
- 7 interconnected services
- Shared persistent storage for static files and Specify6 data
- Redis for inter-service communication
- Nginx as the entry point and static file server

## Prerequisites

- kubectl version 1.29+ (includes Kustomize)
- Access to target Kubernetes cluster
- Environment-specific `.env` files configured

## Quick Start

1. **Clone this repository**:
   ```bash
   git clone <repository-url>
   cd herbarium-specify-kustomize
   ```

2. **Configure environment variables**:
   ```bash
   # Create and edit test environment variables
   cp kustomize/overlays/test/.env.example kustomize/overlays/test/.env
   nano kustomize/overlays/test/.env
   
   # Create and edit production environment variables
   cp kustomize/overlays/prod/.env.example kustomize/overlays/prod/.env
   nano kustomize/overlays/prod/.env
   ```

3. **Deploy to test environment**:
   ```bash
   kubectl apply -k kustomize/overlays/test --namespace herbarium-specify
   ```

4. **Deploy to production**:
   ```bash
   kubectl apply -k kustomize/overlays/prod --namespace herbarium-specify
   ```

## Environment Variables

Each overlay requires a `.env` file with the following variables:

```bash
# Database Configuration
DATABASE_HOST=your-database-host
DATABASE_NAME=your-database-name
DATABASE_USER=your-database-user
DATABASE_PASSWORD=your-database-password

# Application Configuration
SECRET_KEY=your-secret-key
DEBUG=false

# Service URLs
REDIS_URL=redis://redis-service:6379
ASSET_SERVER_URL=http://asset-server-service:8080
ASSET_SERVER_KEY=your-asset-server-key
REPORT_RUNNER_URL=http://report-runner-service:8088

# Specify6 Configuration
SPECIFY6_DIR=/opt/specify6
```

## Deployment Environments

### Test Environment
- **Resources**: Reduced resource limits for cost efficiency
- **Scaling**: Limited horizontal scaling (1-2 replicas)
- **Storage**: Smaller persistent volumes (10Gi attachments, 1Gi redis)
- **Resource Suffix**: `-test`

### Production Environment
- **Resources**: Full resource allocation
- **Scaling**: Full horizontal scaling (up to 10 replicas)
- **Storage**: Production-sized persistent volumes (100Gi attachments, 5Gi redis)
- **Resource Suffix**: `-prod`

## Local Testing

### Prerequisites for Local Testing
- Docker Desktop with Kubernetes enabled, OR
- Minikube installed and running

### Step 1: Validate Configuration (No Cluster Required)

Validate your Kustomize configuration without deploying:

```bash
# Validate base configuration
kubectl kustomize kustomize/base

# Validate test environment (check for errors)
kubectl kustomize kustomize/overlays/test

# Validate production environment
kubectl kustomize kustomize/overlays/prod

# Count resources that will be created
kubectl kustomize kustomize/overlays/test | grep -c "^kind:"
```

### Step 2: Dry Run Deployment (Requires Cluster Connection)

Test deployment without actually creating resources:

```bash
# Dry run for test environment
kubectl apply -k kustomize/overlays/test --dry-run=client

# Dry run with server-side validation (requires cluster access)
kubectl apply -k kustomize/overlays/test --dry-run=server
```

### Step 3: Deploy to Local Kubernetes

**Option A: Docker Desktop Kubernetes**
```bash
# Ensure Docker Desktop Kubernetes is enabled
kubectl config use-context docker-desktop

# Create namespace
kubectl create namespace herbarium-specify

# Deploy test environment
kubectl apply -k kustomize/overlays/test -n herbarium-specify

# Watch deployment progress
kubectl get pods -n herbarium-specify -w
```

**Option B: Minikube**
```bash
# Start Minikube
minikube start

# Create namespace
kubectl create namespace herbarium-specify

# Deploy test environment
kubectl apply -k kustomize/overlays/test -n herbarium-specify

# Watch deployment progress
kubectl get pods -n herbarium-specify -w
```

### Step 4: Verify Deployment

```bash
# Check all resources
kubectl get all -n herbarium-specify

# Check pods are running
kubectl get pods -n herbarium-specify

# Check services
kubectl get svc -n herbarium-specify

# Check persistent volume claims
kubectl get pvc -n herbarium-specify

# Check specific pod logs
kubectl logs -n herbarium-specify -l app=specify-app-deployment --tail=50
```

### Step 5: Test Service Connectivity

```bash
# Port forward to nginx service
kubectl port-forward -n herbarium-specify svc/nginx-service-test 8080:80

# In another terminal, test the connection
curl http://localhost:8080/health

# Or open in browser
open http://localhost:8080
```

### Step 6: Clean Up

```bash
# Delete all resources
kubectl delete -k kustomize/overlays/test -n herbarium-specify

# Or delete the entire namespace
kubectl delete namespace herbarium-specify
```

## Validation

Quick validation commands:

```bash
# Validate base configuration
kubectl kustomize kustomize/base

# Validate test environment
kubectl kustomize kustomize/overlays/test

# Validate production environment
kubectl kustomize kustomize/overlays/prod
```

## Monitoring

Check deployment status:

```bash
# Check all test resources
kubectl get all -l variant=test -n herbarium-specify

# Check all production resources
kubectl get all -l variant=prod -n herbarium-specify

# Check persistent volumes
kubectl get pvc -n herbarium-specify
```

## Service Communication

The services communicate internally using Kubernetes DNS:

- **Nginx** → **Specify App** (port 8000)
- **Specify App** → **Redis** (port 6379)
- **Specify App** → **Asset Server** (port 8080)
- **Specify App** → **Report Runner** (port 8088)
- **Specify Worker** → **Redis** (port 6379)
- **All services** → **Specify6** (shared volume)

## Migration from Docker Compose

This configuration migrates from the original Docker Compose setup. See `docker-compose-reference.yml` for the original configuration and mapping to Kubernetes resources.

## Organizational Standards

This configuration follows organizational Kustomize standards:
- Base/overlays structure
- Strategic merge patches
- secretGenerator for environment variables
- configMapGenerator for configuration files
- Consistent labeling with `variant` labels

## Support

For issues or questions:
1. Check the `kustomize/README.md` for detailed Kustomize usage
2. Review the validation commands above
3. Consult the organizational Kustomize tutorial: https://github.com/dbca-wa/kustomize-tutorial

## References

- [Kubernetes Kustomize Documentation](https://kubernetes.io/docs/tasks/manage-kubernetes-objects/kustomization/)
- [Kustomize GitHub Repository](https://github.com/kubernetes-sigs/kustomize)
- [Organizational Kustomize Tutorial](https://github.com/dbca-wa/kustomize-tutorial)