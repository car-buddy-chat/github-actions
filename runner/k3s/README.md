# Self-Hosted GitHub Actions Runner on k3s

This directory contains Kubernetes manifests for running GitHub Actions self-hosted runners on a k3s cluster.

## Overview

The setup uses GitHub Actions Runner Controller (ARC) to automatically create and manage runner pods based on job demand.

## Architecture

```
GitHub Actions
    ↓
ARC Controller (in-cluster)
    ↓
Ephemeral Runner Pods
    ├── GitHub Runner Agent
    ├── Docker (via DinD sidecar)
    └── kubectl (for k8s interactions)
```

## Prerequisites

- k3s cluster with sufficient resources (min 4GB memory available)
- kubectl access with admin permissions
- GitHub Personal Access Token (PAT) with `repo`, `workflow` scopes
- Helm 3.x (for ARC installation)

## Installation

### 1. Install Actions Runner Controller

```bash
helm repo add actions-runner-controller https://actions-runner-controller.github.io/actions-runner-controller
helm repo update

helm install arc actions-runner-controller/actions-runner-controller \
  --namespace arc-systems \
  --create-namespace \
  --set githubWebhookServer.enabled=true
```

### 2. Create GitHub PAT Secret

```bash
kubectl create secret generic controller-manager \
  --from-literal=github_token=ghp_xxxxxxxxxxxxxxxxxxxx \
  -n arc-systems
```

### 3. Apply Runner Configuration

```bash
kubectl apply -f deployment.yaml
```

## Configuration

Edit `deployment.yaml` to customize:
- Runner name and group
- Resource requests/limits
- Docker-in-Docker settings
- Environment variables

## Docker-in-Docker

The runner includes a DinD sidecar for building container images. See [../docker-in-docker/README.md](../docker-in-docker/README.md).

## Monitoring

```bash
# View runner pods
kubectl get pods -n arc-runners

# View runner logs
kubectl logs -n arc-runners <pod-name>

# View ARC controller logs
kubectl logs -n arc-systems deployment/arc
```

## Troubleshooting

**Runners not registering:**
```bash
kubectl describe pod -n arc-runners <pod-name>
```

**Docker operations failing:**
Check DinD sidecar is running:
```bash
kubectl logs -n arc-runners <pod-name> -c docker-daemon
```

**Runner runs failing:**
- Check ECR credentials are configured
- Verify runner has network access to registries
- Check security group rules

## Cost Optimization

- Scale runners down to 0 during off-hours
- Use smaller resource requests for light workloads
- Monitor actual vs requested resources

## Security

- Runners use minimal service accounts with only required permissions
- ECR credentials via IAM role (recommended) or Secrets Manager
- Network policies restrict egress to necessary endpoints
- All runner pods are ephemeral and discarded after use

## References

- [ARC Documentation](https://github.com/actions/actions-runner-controller)
- [GitHub Runner Specifications](https://docs.github.com/en/actions/hosting-your-own-runners/about-self-hosted-runners)
