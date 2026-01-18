# Reusable GitHub Actions

This repository contains reusable GitHub Actions that can be imported into any workflow using:

```yaml
uses: car-buddy-chat/github-actions/actions/action-name@main
```

## Available Actions

### build-image

Builds and pushes Docker images to ECR or other registries.

**Usage:**
```yaml
- uses: car-buddy-chat/github-actions/actions/build-image@main
  with:
    registry: ${{ steps.login-ecr.outputs.registry }}
    repository: my-app
```

See [actions/build-image/README.md](../actions/build-image/README.md) for full details.

### deploy-k3s

Deploys to k3s cluster via Argo CD webhook.

(Documentation coming soon)

### notify-slack

Sends Slack notifications with job status.

(Documentation coming soon)

## Creating New Actions

1. Create a directory in `actions/`
2. Add `action.yml` with metadata
3. Add implementation (shell script, composite steps, or container)
4. Add README.md with usage examples
5. Test thoroughly before merging
