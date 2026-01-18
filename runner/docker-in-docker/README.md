# Docker-in-Docker for GitHub Actions Runners

This directory contains the Docker-in-Docker (DinD) configuration for building container images in runner pods.

## Overview

Docker-in-Docker allows the GitHub Actions runner to:
- Build Docker images using `docker build`
- Push images to ECR or other registries
- Run containerized tests

## Implementation

The DinD service is deployed as:
1. A separate Kubernetes service with volume storage
2. Sidecar access from runner pods via TCP socket
3. Uses docker:26-dind image for security and performance

## Configuration Files

- `deployment.yaml` - DinD service deployment
- Configured for socket access at `docker-daemon:2375`

## Usage in Workflows

The runner automatically connects to DinD via:
```bash
export DOCKER_HOST=tcp://docker-daemon:2375
docker build ...
docker push ...
```

## Resource Usage

- Typical memory: 256-512Mi
- Temporary storage: 5-10Gi (configurable)
- Cleanup happens automatically after pod termination

## Security Considerations

- DinD runs with `privileged: true` (required for container operations)
- No TLS required (development setup)
- For production, add TLS and authentication
- Limit network access via NetworkPolicy

## Monitoring

Monitor DinD health:
```bash
kubectl logs -n docker-daemon deployment/dind-daemon
kubectl exec -n docker-daemon <pod> -- docker version
```
