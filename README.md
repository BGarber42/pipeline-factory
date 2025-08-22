# Pipeline Factory

A centralized, secure set of GitHub reusable workflows and composite actions that hundreds of services can consume to lint, test, scan, build, push, and optionally deploy containerized apps — without managing cloud secrets.

## How it works

- **OIDC Authentication**: Uses GitHub OIDC to assume AWS roles (no long-lived credentials)
- **Security Scanning**: CodeQL SAST and Trivy scans enforced (fail on HIGH/CRITICAL)
- **Container Build**: Build and push image to ECR with optional ECS deployment
- **Reusable Components**: Composite actions for common operations

## Repository Structure

```
pipeline-factory/
  .github/
    actions/
      aws-oidc-credentials/
        action.yml
      trivy-scan/
        action.yml
    workflows/
      reusable-aws-cicd.yml
    dependabot.yml
  README.md
```

## Inputs

| Input | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `aws-role-to-assume` | string | ✅ | - | ARN of IAM role to assume via OIDC |
| `aws-region` | string | ❌ | `us-east-1` | AWS region for operations |
| `ecr-repository-name` | string | ✅ | - | Target ECR repository name |
| `dockerfile-path` | string | ❌ | `./Dockerfile` | Path to Dockerfile |
| `context` | string | ❌ | `.` | Docker build context |
| `image-tag` | string | ❌ | - | Optional explicit image tag (defaults to short SHA) |
| `codeql-languages` | string | ❌ | `javascript,python,go` | Comma-separated list of CodeQL languages |
| `test-command` | string | ❌ | `''` | Optional test command to run |
| `enable-ecs-deploy` | boolean | ❌ | `false` | If true, update ECS service to new image |
| `ecs-cluster` | string | ❌ | - | ECS cluster name (required if deploy enabled) |
| `ecs-service` | string | ❌ | - | ECS service name (required if deploy enabled) |
| `ecs-container-name` | string | ❌ | - | Container name in task definition (required if deploy enabled) |

## Outputs

| Output | Description |
|--------|-------------|
| `image_tag` | Final image tag used |
| `image_uri` | ECR image URI with tag (`account.dkr.ecr.region.amazonaws.com/repo:tag`) |
| `image_digest` | Pushed image digest |

## Secrets

- Callers use `secrets: inherit` to pass any needed repo/org secrets
- No static AWS secrets required (OIDC only)

## Usage Example

```yaml
name: CI/CD

on:
  push:
    branches: [ main ]

jobs:
  build-and-deploy:
    uses: your-github-org/pipeline-factory/.github/workflows/reusable-aws-cicd.yml@v1
    with:
      aws-role-to-assume: arn:aws:iam::123456789012:role/PipelineFactoryDeployerRole
      aws-region: us-east-1
      ecr-repository-name: my-web-app
      dockerfile-path: ./Dockerfile
      context: .
      test-command: 'npm ci && npm test'
      enable-ecs-deploy: false
    secrets: inherit
```

### Deploy to ECS

To enable ECS deployment, add these parameters:

```yaml
with:
  enable-ecs-deploy: true
  ecs-cluster: my-cluster
  ecs-service: my-service
  ecs-container-name: web
```

## Security Posture

- **No long-lived credentials**: Uses OIDC for AWS authentication
- **Mandatory security scans**: CodeQL SAST and Trivy container scans
- **Fail on critical issues**: Build fails on HIGH/CRITICAL security findings
- **Standardized pipeline**: Reduces configuration drift across services

## Prerequisites

- GitHub organization with Actions enabled
- AWS account with:
  - OIDC identity provider configured for `token.actions.githubusercontent.com`
  - IAM role permitting ECR push and optional ECS deploy
- ECR repository per service or permissions to create them

## Versioning

- Create release tags after testing (e.g., `v1.0.0`)
- Create moving major tags `v1` pointing to latest stable release
- Callers should pin to `@v1` (or commit SHA for stricter pinning)

## Contributing

1. Make changes to workflows or actions
2. Test thoroughly with sample applications
3. Create a release tag
4. Update the moving major tag

## License

[Add your license here]
