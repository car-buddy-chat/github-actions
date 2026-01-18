# GitHub Actions Repository

Centralized repository for:
- **Reusable GitHub Actions** - Composite and container actions used across car-buddy-chat projects
- **Reusable Workflows** - Workflow templates for CI/CD, testing, deployment
- **Self-Hosted Runner Configurations** - Kubernetes manifests and configuration for self-hosted runners on k3s
- **CI/CD Best Practices** - Documented patterns and security configurations

## Directory Structure

```
.
├── .github/
│   └── workflows/          # Template workflows for importing
├── actions/                # Reusable composite/container actions
│   ├── build-image/        # Build Docker images with Buildah/Docker
│   ├── deploy-k3s/         # Deploy to k3s via Argo CD
│   └── notify-slack/       # Send Slack notifications
├── workflows/              # Reusable workflow templates
│   ├── build-and-push.yml
│   ├── test-suite.yml
│   └── deploy-k3s.yml
├── runner/                 # Self-hosted runner configurations
│   ├── k3s/                # k3s cluster runner setup
│   │   ├── dockerfile      # Runner pod container image
│   │   ├── deployment.yaml # Kubernetes deployment manifest
│   │   └── README.md       # Setup instructions
│   └── docker-in-docker/   # DinD service configuration
│       ├── deployment.yaml
│       └── README.md
├── docs/                   # Documentation
│   ├── reusable-actions.md
│   ├── reusable-workflows.md
│   └── runner-setup.md
└── README.md               # This file
```

## Quick Start

### Using Reusable Actions

```yaml
name: Deploy
on: [push]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Build Docker image
        uses: car-buddy-chat/github-actions/actions/build-image@main
        with:
          registry: ${{ steps.login-ecr.outputs.registry }}
          repository: my-app
          
      - name: Deploy to k3s
        uses: car-buddy-chat/github-actions/actions/deploy-k3s@main
        with:
          host: 34.252.7.204
          namespace: production
```

### Using Reusable Workflows

```yaml
name: Full CI/CD
on: [push, pull_request]

jobs:
  call-build:
    uses: car-buddy-chat/github-actions/.github/workflows/build-and-push.yml@main
    with:
      registry: ecr
      
  call-deploy:
    needs: call-build
    uses: car-buddy-chat/github-actions/.github/workflows/deploy-k3s.yml@main
    with:
      host: 34.252.7.204
```

## Current Projects Using This

- **carbuddy-uk** - Website deployment via GitHub Actions → ECR → k3s
- **carbuddy-webapp** - Frontend build and deployment
- *More projects to be added*

## Running Self-Hosted Runners on k3s

See [runner/k3s/README.md](runner/k3s/README.md) for:
- Setting up GitHub Actions runner controller on k3s
- Docker-in-Docker service configuration
- Permissions and networking setup

## Documentation

- [Reusable Actions Guide](docs/reusable-actions.md)
- [Reusable Workflows Guide](docs/reusable-workflows.md)
- [Self-Hosted Runner Setup](docs/runner-setup.md)

## Contributing

When adding new actions or workflows:
1. Create action in `actions/` or workflow in `workflows/`
2. Add comprehensive documentation
3. Test in your own project first
4. Create PR with examples

## Security

- GitHub Secrets are managed per-repository
- ECR credentials stored in AWS Secrets Manager
- Self-hosted runners use least-privilege service accounts
- See [SECURITY.md](SECURITY.md) for detailed guidelines

## Maintenance

- Review and update runner configurations quarterly
- Keep GitHub Actions versions current
- Monitor for security advisories
- Update documentation when patterns change

