# Container Signing Template Guide

## Overview

This guide explains how to use the RHTAS GitLab CI template for container image signing in your pipelines.

## Template Location

```yaml
include:
  - remote: 'https://raw.githubusercontent.com/ihsanmokhlisse/rhtas-gitlab-ci-templates/main/templates/container-signing.yml'
```

## Prerequisites

1. RHTAS (Red Hat Trusted Artifact Signer) instance running and accessible
2. GitLab project with CI/CD pipelines enabled
3. Container registry credentials configured
4. OIDC authentication set up between GitLab and RHTAS

## Template Parameters

| Parameter | Description | Required | Default |
|-----------|-------------|----------|---------|
| `RHTAS_URL` | URL to RHTAS Fulcio service | Yes | - |
| `REKOR_URL` | URL to RHTAS Rekor service | Yes | - |
| `CONTAINER_IMAGE` | Container image to sign (including tag) | Yes | - |
| `REGISTRY_USER` | Container registry username | Yes | - |
| `REGISTRY_PASSWORD` | Container registry password | Yes | - |
| `COSIGN_VERSION` | Version of cosign to use | No | `v2.1.1` |
| `OIDC_TOKEN_FILE` | Path to OIDC token file | No | `.oidc-token` |
| `EXTRA_ANNOTATIONS` | Additional annotations to add to signature | No | - |
| `SIGN_OUTPUT_FILE` | File to store signature information | No | `signature.json` |

## Basic Usage

```yaml
stages:
  - build
  - sign

build-image:
  stage: build
  script:
    - docker build -t registry.example.com/myproject/myapp:latest .
    - docker push registry.example.com/myproject/myapp:latest

sign-container:
  stage: sign
  extends: .container-signing
  variables:
    RHTAS_URL: https://fulcio.rhtas.example.com
    REKOR_URL: https://rekor.rhtas.example.com
    CONTAINER_IMAGE: registry.example.com/myproject/myapp:latest
    REGISTRY_USER: $REGISTRY_USERNAME
    REGISTRY_PASSWORD: $REGISTRY_PASSWORD
```

## Advanced Configuration

### Custom Annotations

```yaml
sign-container:
  stage: sign
  extends: .container-signing
  variables:
    RHTAS_URL: https://fulcio.rhtas.example.com
    REKOR_URL: https://rekor.rhtas.example.com
    CONTAINER_IMAGE: registry.example.com/myproject/myapp:latest
    REGISTRY_USER: $REGISTRY_USERNAME
    REGISTRY_PASSWORD: $REGISTRY_PASSWORD
    EXTRA_ANNOTATIONS: |
      build-date=$(date -u +'%Y-%m-%dT%H:%M:%SZ')
      git-commit=$CI_COMMIT_SHA
      pipeline-id=$CI_PIPELINE_ID
```

### Multiple Container Images

```yaml
sign-containers:
  stage: sign
  parallel:
    matrix:
      - IMAGE_NAME: [
          "registry.example.com/myproject/app1:latest",
          "registry.example.com/myproject/app2:latest"
        ]
  extends: .container-signing
  variables:
    RHTAS_URL: https://fulcio.rhtas.example.com
    REKOR_URL: https://rekor.rhtas.example.com
    CONTAINER_IMAGE: $IMAGE_NAME
    REGISTRY_USER: $REGISTRY_USERNAME
    REGISTRY_PASSWORD: $REGISTRY_PASSWORD
```

## Template Implementation Details

### Authentication

The template uses OIDC authentication to authenticate with RHTAS. GitLab generates an OIDC token that includes claims about the pipeline, project, and user. RHTAS verifies this token against the configured OIDC provider.

```yaml
.container-signing:
  script:
    - echo "$CI_JOB_JWT_V2" > $OIDC_TOKEN_FILE
    - export COSIGN_EXPERIMENTAL=1
    - cosign login $REGISTRY_URL -u $REGISTRY_USER -p $REGISTRY_PASSWORD
    - cosign sign --oidc-issuer $CI_SERVER_URL --identity-token $OIDC_TOKEN_FILE $CONTAINER_IMAGE
```

### Signature Verification

After signing, the template verifies that the signature was successfully created and can be validated:

```yaml
.container-signing:
  script:
    # ... signing steps ...
    - cosign verify --certificate-identity $CI_PROJECT_PATH --certificate-oidc-issuer $CI_SERVER_URL $CONTAINER_IMAGE
```

## Integrating with CI/CD Workflows

### Before Build Stage (Prepare)

```yaml
prepare:
  stage: prepare
  script:
    - cosign version # Verify cosign is available
    - env | grep CI_ # Show GitLab CI environment variables
  allow_failure: false
```

### After Sign Stage (Verify)

```yaml
verify-signature:
  stage: verify
  script:
    - cosign verify --certificate-identity $CI_PROJECT_PATH --certificate-oidc-issuer $CI_SERVER_URL $CONTAINER_IMAGE
  needs:
    - sign-container
```

## Security Considerations

1. **Credentials Handling**: Use GitLab CI/CD variables for sensitive information
2. **OIDC Configuration**: Ensure proper OIDC claim verification in RHTAS
3. **Signature Verification**: Always verify signatures after signing
4. **Token Permissions**: Limit OIDC token permissions to only what's needed

## Troubleshooting

### Common Issues

1. **Authentication failures**:
   - Verify OIDC provider configuration
   - Check GitLab project settings for JWT generation
   - Validate RHTAS policies

2. **Registry access issues**:
   - Verify registry credentials
   - Check network connectivity to registry
   - Ensure proper permissions for the image

3. **Signing failures**:
   - Check RHTAS logs for errors
   - Verify connectivity to RHTAS services
   - Validate image exists in registry

### Debugging Steps

```yaml
sign-container:
  stage: sign
  extends: .container-signing
  variables:
    RHTAS_URL: https://fulcio.rhtas.example.com
    REKOR_URL: https://rekor.rhtas.example.com
    CONTAINER_IMAGE: registry.example.com/myproject/myapp:latest
    REGISTRY_USER: $REGISTRY_USERNAME
    REGISTRY_PASSWORD: $REGISTRY_PASSWORD
  before_script:
    - export COSIGN_DEBUG=1
    - curl -s $RHTAS_URL/api/v1/rootCert # Test connectivity
    - curl -s $REKOR_URL/api/v1/log/info # Test connectivity
```

## Example Complete Pipeline

```yaml
include:
  - remote: 'https://raw.githubusercontent.com/ihsanmokhlisse/rhtas-gitlab-ci-templates/main/templates/container-signing.yml'

stages:
  - build
  - test
  - sign
  - deploy

variables:
  DOCKER_HOST: tcp://docker:2375
  DOCKER_DRIVER: overlay2
  REGISTRY: registry.example.com
  IMAGE_NAME: myproject/myapp
  IMAGE_TAG: $CI_COMMIT_SHORT_SHA

services:
  - docker:dind

build-image:
  stage: build
  image: docker:latest
  script:
    - docker build -t $REGISTRY/$IMAGE_NAME:$IMAGE_TAG .
    - docker push $REGISTRY/$IMAGE_NAME:$IMAGE_TAG

test-image:
  stage: test
  image: $REGISTRY/$IMAGE_NAME:$IMAGE_TAG
  script:
    - /app/healthcheck.sh

sign-container:
  stage: sign
  extends: .container-signing
  variables:
    RHTAS_URL: https://fulcio.rhtas.example.com
    REKOR_URL: https://rekor.rhtas.example.com
    CONTAINER_IMAGE: $REGISTRY/$IMAGE_NAME:$IMAGE_TAG
    REGISTRY_USER: $REGISTRY_USERNAME
    REGISTRY_PASSWORD: $REGISTRY_PASSWORD
    EXTRA_ANNOTATIONS: |
      build-date=$(date -u +'%Y-%m-%dT%H:%M:%SZ')
      git-commit=$CI_COMMIT_SHA
      pipeline-id=$CI_PIPELINE_ID

deploy:
  stage: deploy
  script:
    - cosign verify --certificate-identity $CI_PROJECT_PATH --certificate-oidc-issuer $CI_SERVER_URL $REGISTRY/$IMAGE_NAME:$IMAGE_TAG
    - echo "Deploying verified image $REGISTRY/$IMAGE_NAME:$IMAGE_TAG"
    - kubectl set image deployment/myapp myapp=$REGISTRY/$IMAGE_NAME:$IMAGE_TAG
```

## Support and Resources

- [RHTAS Documentation](https://docs.rhtas.example.com)
- [GitLab CI/CD Documentation](https://docs.gitlab.com/ee/ci/)
- [Cosign Documentation](https://github.com/sigstore/cosign)
- Maintainer Contact: Ihsan Mokhlisse <imokhlis@redhat.com>