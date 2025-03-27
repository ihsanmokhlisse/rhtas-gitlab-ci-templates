# GitLab CI Artifact Signing Guide

## Overview

This guide details how to implement artifact signing in your GitLab CI/CD pipelines using RHTAS templates.

## Prerequisites

- GitLab CI/CD environment
- Access to RHTAS instance
- Required environment variables configured
- Alpine-based runner image

## Base Configuration

### Template Inclusion

```yaml
include:
  - local: 'gitlab-ci/templates/rhtas-base.gitlab-ci.yml'
  - local: 'gitlab-ci/templates/secure-signing.gitlab-ci.yml'
```

### Environment Setup

```yaml
variables:
  # RHTAS URLs
  TUF_URL: https://tuf-trusted-artifact-signer.${APPS_DOMAIN}
  COSIGN_FULCIO_URL: https://fulcio-server-trusted-artifact-signer.${APPS_DOMAIN}
  COSIGN_REKOR_URL: https://rekor-server-trusted-artifact-signer.${APPS_DOMAIN}
  
  # Security Settings
  COSIGN_EXPERIMENTAL: "1"
  COSIGN_DEBUG: "1"
  COSIGN_TIMEOUT: "300s"
  
  # OIDC Configuration
  COSIGN_OIDC_ISSUER: ${GITLAB_URL}
  COSIGN_CERTIFICATE_OIDC_ISSUER: ${GITLAB_URL}
```

## Pipeline Implementation

### Basic Signing Pipeline

```yaml
stages:
  - build
  - sign
  - verify
  - deploy

build:
  stage: build
  script:
    - mvn clean package
  artifacts:
    paths:
      - target/*.jar

sign:
  stage: sign
  extends:
    - .rhtas-base
    - .rhtas-setup
  script:
    - verify_signature "${ARTIFACT_PATH}" "${ARTIFACT_PATH}.bundle"
  dependencies:
    - build

verify:
  stage: verify
  extends:
    - .rhtas-base
  script:
    - verify_signature "${ARTIFACT_PATH}"
  dependencies:
    - sign
```

### Advanced Configuration

#### Identity Verification

```yaml
.verify-identity:
  script:
    - verify_signature "${ARTIFACT_PATH}" "${ARTIFACT_PATH}.bundle"
    - verify_certificate_claims
    - verify_timestamp
```

#### Secure Cleanup

```yaml
.cleanup:
  after_script:
    - secure_cleanup "artifacts"
    - remove_sensitive_files
```

## Security Features

### OIDC Integration

```yaml
id_tokens:
  SIGSTORE_ID_TOKEN:
    aud: ${COSIGN_OIDC_CLIENT_ID}
```

### Protected Variables

```yaml
variables:
  NEXUS_PASSWORD:
    value: "secret"
    protected: true
    masked: true
```

## Error Handling

### Retry Mechanism

```yaml
.retry-logic:
  retry:
    max: 3
    when:
      - runner_system_failure
      - stuck_or_timeout_failure
```

### Validation Checks

```yaml
verify_environment() {
  for var in APPS_DOMAIN GITLAB_URL COSIGN_CLIENT_ID; do
    if [ -z "${!var}" ]; then
      echo "Error: Required variable $var is not set"
      exit 1
    fi
  done
}
```

## Best Practices

1. **Security**:
   - Use protected variables
   - Implement secure cleanup
   - Validate all operations

2. **Error Handling**:
   - Implement retry mechanisms
   - Provide clear error messages
   - Handle failures gracefully

3. **Pipeline Design**:
   - Separate build and sign stages
   - Include verification steps
   - Maintain audit trail

## Troubleshooting

### Common Issues

1. **Tool Installation**:
   - Verify Alpine package availability
   - Check network connectivity
   - Validate tool versions

2. **OIDC Configuration**:
   - Verify GitLab URL
   - Check client ID configuration
   - Validate token permissions

3. **Signing Operations**:
   - Check Fulcio availability
   - Verify Rekor connectivity
   - Validate artifact paths

### Debug Mode

Enable detailed logging:
```yaml
variables:
  COSIGN_DEBUG: "1"
  COSIGN_EXPERIMENTAL: "1"
```

## Support

For pipeline issues, contact:
- Maintainer: Ihsan Mokhlisse <imokhlis@redhat.com>