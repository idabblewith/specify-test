# Specify Kubernetes Configuration

This directory contains the Kustomize configuration for deploying the Specify application stack to Kubernetes following organizational standards.

## Directory Structure

```
kustomize/
├── base/                           # Base Kubernetes resources
│   ├── kustomization.yaml         # Base configuration
│   ├── deployment.yaml            # All deployment definitions
│   ├── deployment_hpa.yaml        # Horizontal Pod Autoscaler configurations
│   └── service.yaml               # All service definitions
└── overlays/                      # Environment-specific configurations
    ├── test/                      # Test environment
    │   ├── kustomization.yaml     # Test overlay configuration
    │   ├── .env                   # Environment variables (gitignored)
    │   ├── default.conf           # Nginx configuration
    │   ├── pvcs.yaml              # Persistent Volume Claims
    │   ├── deployment_patch.yaml  # Deployment patches
    │   └── deployment_hpa_patch.yaml # HPA patches
    └── prod/                      # Production environment
        ├── kustomization.yaml     # Production overlay configuration
        ├── .env                   # Environment variables (gitignored)
        ├── default.conf           # Nginx configuration
        ├── pvcs.yaml              # Persistent Volume Claims
        ├── deployment_patch.yaml  # Deployment patches
        └── deployment_hpa_patch.yaml # HPA patches
```

## Prerequisites

- kubectl version 1.29+ with integrated Kustomize support
- Access to target Kubernetes cluster
- Environment-specific .env files (see templates)

## Setup

1. **Configure environment files**: Edit the `.env` files in each overlay directory with your environment-specific values:
   ```bash
   # Edit test environment variables
   nano kustomize/overlays/test/.env
   
   # Edit production environment variables
   nano kustomize/overlays/prod/.env
   ```

2. **Configure kubectl context**: Ensure you're connected to the correct cluster:
   ```bash
   kubectl config current-context
   kubectl cluster-info
   ```

## Usage

### Validate Configuration
```bash
# Validate base configuration
kubectl kustomize kustomize/base

# Validate environment-specific configuration
kubectl kustomize kustomize/overlays/test
kubectl kustomize kustomize/overlays/prod
```

### Deploy to Environment
```bash
# Deploy to test
kubectl apply -k kustomize/overlays/test

# Deploy to production
kubectl apply -k kustomize/overlays/prod
```

### Verify Deployment
```bash
# Check pods
kubectl get pods -n herbarium-specify

# Check services
kubectl get services -n herbarium-specify

# Check ingress
kubectl get ingress -n herbarium-specify

# Check persistent volumes
kubectl get pvc -n herbarium-specify
```

## Organizational Standards

This configuration follows organizational Kustomize standards:

- **Labels**: All resources include `businessUnit: OIM` label
- **Base/Overlays Structure**: Common resources in base, environment-specific patches in overlays
- **Secret Management**: Uses secretGenerator with .env files (gitignored)
- **Naming**: Consistent naming with environment suffixes (-dev, -uat, -prod)

## Environment Variables

Each overlay requires a `.env` file with the following variables:

- `DATABASE_HOST`: Database server hostname
- `DATABASE_NAME`: Database name
- `DATABASE_USER`: Database username
- `DATABASE_PASSWORD`: Database password
- `SECRET_KEY`: Django secret key
- `DEBUG`: Debug mode (true/false)
- `REDIS_URL`: Redis connection URL
- `ASSET_SERVER_URL`: Asset server URL
- `ASSET_SERVER_KEY`: Asset server authentication key
- `REPORT_RUNNER_URL`: Report runner URL
- `SPECIFY6_DIR`: Specify6 directory path

## Troubleshooting

### Common Issues

1. **Missing .env file**: Ensure .env files are created from templates
2. **Invalid context**: Verify kubectl is connected to correct cluster
3. **Resource conflicts**: Check for existing resources with same names
4. **Storage issues**: Verify storage classes are available in cluster

### Validation Commands
```bash
# Check Kustomize syntax
kubectl kustomize kustomize/overlays/development --dry-run=client

# Validate against cluster
kubectl apply -k kustomize/overlays/development --dry-run=server
```

For more information on organizational Kustomize patterns, see: https://github.com/dbca-wa/kustomize-tutorial