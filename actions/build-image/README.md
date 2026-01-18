# Build Docker Image Action

Builds and pushes a Docker image to a container registry (ECR, DockerHub, etc).

## Usage

```yaml
- uses: car-buddy-chat/github-actions/actions/build-image@main
  with:
    registry: 476114155266.dkr.ecr.eu-west-1.amazonaws.com
    repository: my-app
    image-tag: v1.0.0
```

## Inputs

- `registry` - Container registry URL (required)
- `repository` - Repository name (required)
- `image-tag` - Image tag, defaults to github.sha
- `dockerfile` - Path to Dockerfile, defaults to ./Dockerfile

## Outputs

- `image-uri` - Full image URI for use in subsequent steps

## Requirements

- Docker must be available in the runner environment
- Registry credentials should be configured prior to this action
