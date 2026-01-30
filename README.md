# Port.io Kubernetes Deployment Demo

A demonstration project showcasing automated Kubernetes deployments integrated with [Port.io](https://getport.io), an internal developer portal platform.

## Overview

This project provides GitHub Actions workflows that enable self-service Kubernetes deployments and rollbacks, with full integration into Port.io for tracking, logging, and visibility.

## Features

- **Self-Service Deployments**: Deploy services to staging or production environments via GitHub Actions
- **Rollback Capability**: Quickly rollback to previous versions in case of issues
- **Port.io Integration**: Automatic tracking of deployments, versions, and application state in Port
- **Demo Cluster Setup**: Automated creation of a kind-based Kubernetes cluster for testing
- **Health Verification**: Built-in smoke tests and deployment health checks
- **Detailed Logging**: Comprehensive deployment summaries and status updates

## Prerequisites

- GitHub repository with Actions enabled
- Kubernetes cluster (or use the demo cluster setup)
- Port.io account with API credentials
- kubectl configured for your cluster

## Setup

### 1. Configure GitHub Secrets

Add the following secrets to your GitHub repository:

```
PORT_CLIENT_ID       - Your Port.io client ID
PORT_CLIENT_SECRET   - Your Port.io client secret
KUBECONFIG          - Base64-encoded kubeconfig file for cluster access
DOCKER_REGISTRY     - (Optional) Your Docker registry URL
```

To encode your kubeconfig:
```bash
cat ~/.kube/config | base64 | pbcopy
```

### 2. Port.io Blueprints

Ensure the following blueprints exist in your Port.io workspace:

- `application` - Represents deployed applications
- `k8sDeployment` - Represents individual Kubernetes deployments
- `namespace` - Represents Kubernetes namespaces

### 3. Demo Cluster (Optional)

To create a local demo cluster with sample services:

1. Go to **Actions** → **Setup Demo Kubernetes Cluster**
2. Select `create` action
3. Run the workflow

This creates a kind cluster with:
- 1 control plane node + 2 worker nodes
- Production and staging namespaces
- Sample services: `payment-service` and `search-service`

## Usage

### Deploy a Service

1. Navigate to **Actions** → **Deploy Service**
2. Provide the following inputs:
   - **Service Name**: Name of your Kubernetes deployment (e.g., `payment-service`)
   - **Version**: Version tag to deploy (e.g., `v3.2.0`)
   - **Environment**: `staging` or `production`
   - **Namespace**: (Optional) Custom namespace, defaults to environment name
   - **Port Run ID**: (Optional) Port.io action run ID for tracking

3. Click **Run workflow**

The deployment will:
- Update the container image
- Wait for rollout completion
- Run smoke tests
- Update Port.io with deployment status
- Verify pod health

### Rollback a Deployment

1. Navigate to **Actions** → **Rollback Service Deployment**
2. Provide the following inputs:
   - **Service Name**: Name of the deployment to rollback
   - **Target Version**: Version to rollback to (e.g., `v3.1.9`)
   - **Namespace**: Kubernetes namespace
   - **Incident ID**: (Optional) Associated incident for tracking
   - **Port Run ID**: (Optional) Port.io action run ID

3. Click **Run workflow**

The rollback will:
- Restore the specified version
- Wait for rollout completion
- Verify deployment health
- Update Port.io with rollback status

## Project Structure

```
.
├── deploy-service.yml           # GitHub Actions workflow for deploying services
├── rollback-deployment.yml      # GitHub Actions workflow for rollbacks
├── setup-demo-cluster.yml       # GitHub Actions workflow for demo cluster setup
└── templates/
    ├── deploy-template.yml      # Template for deployment configurations
    └── rollback-template.yml    # Template for rollback configurations
```

## Workflow Details

### Deploy Service Workflow

The deployment process includes:
1. Image preparation (uses nginx for demo purposes)
2. Kubernetes context configuration
3. Deployment update with new image
4. Rollout status monitoring
5. Health verification and smoke tests
6. Port.io entity updates

### Rollback Workflow

The rollback process includes:
1. Current deployment status check
2. Image rollback to specified version
3. Rollout monitoring
4. Health verification
5. Port.io status updates

## Port.io Integration

This demo automatically:
- Creates/updates application entities with current version and deployment status
- Creates k8sDeployment entities for each deployment
- Links deployments to applications and namespaces
- Logs deployment progress and results to Port.io action runs

## Demo Notes

For demonstration purposes, this project uses `nginx` images with version tags. In a production environment, you would:
- Build and push your actual service container images
- Use a proper container registry (ACR, ECR, GCR, etc.)
- Implement additional security scanning and compliance checks
- Add integration tests and monitoring

## Troubleshooting

### Deployment Fails with "deployment not found"

Ensure the deployment exists in the target namespace:
```bash
kubectl get deployments -n <namespace>
```

### Connection to Kubernetes Fails

Verify your KUBECONFIG secret is correctly base64-encoded and has proper cluster access.

### Port.io Updates Not Working

Check that `PORT_CLIENT_ID` and `PORT_CLIENT_SECRET` are configured and that the blueprints exist in your Port workspace.

## Contributing

This is a demo project for illustration purposes. Feel free to adapt it to your needs.

## License

MIT

## Resources

- [Port.io Documentation](https://docs.getport.io)
- [Kubernetes Documentation](https://kubernetes.io/docs)
- [GitHub Actions Documentation](https://docs.github.com/en/actions)
