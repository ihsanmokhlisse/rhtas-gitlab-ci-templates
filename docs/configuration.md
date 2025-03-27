# RHTAS GitLab CI Configuration Guide

## Variables

### Required Variables

- `RHTAS_URL`: Your RHTAS instance URL
- `RHTAS_OIDC_ISSUER`: GitLab instance URL
- `ARTIFACT_PATH`/`CONTAINER_IMAGE`: Path/URL to sign

### Optional Variables

- `RHTAS_NAMESPACE`: RHTAS installation namespace
- `SIGN_TIMEOUT`: Signing timeout in seconds
- `VERIFICATION_POLICY`: Signature verification policy

## Template Customization

### Artifact Signing

```yaml
sign-jar:
  extends: .rhtas-sign-artifact
  variables:
    ARTIFACT_PATH: "target/*.jar"
```

### Container Signing

```yaml
sign-image:
  extends: .rhtas-sign-container
  variables:
    CONTAINER_IMAGE: "${CI_REGISTRY_IMAGE}:${CI_COMMIT_TAG}"
```

### Verification

```yaml
verify-signature:
  extends: .rhtas-verify
  variables:
    TARGET_PATH: "${CI_REGISTRY_IMAGE}:${CI_COMMIT_TAG}"
```

## Advanced Configuration

- Custom signing policies
- Multiple artifact signing
- CI/CD integration patterns