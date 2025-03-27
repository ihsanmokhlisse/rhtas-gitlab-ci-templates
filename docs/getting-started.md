# Getting Started with RHTAS GitLab CI Templates

This guide helps you integrate Red Hat Trusted Artifact Signer (RHTAS) into your GitLab CI/CD pipelines.

## Prerequisites

- GitLab CI/CD environment
- RHTAS instance (v1.1.1+)
- OpenShift cluster (4.13+) running RHTAS
- GitLab OIDC provider configured in RHTAS

## Quick Setup

1. Configure OIDC in GitLab:
```yaml
oidc:
  enabled: true
  issuer: https://your-gitlab.com
```

2. Configure RHTAS OIDC provider:
```yaml
apiVersion: trusted.signing/v1alpha1
kind: OIDCProvider
metadata:
  name: gitlab-provider
spec:
  issuer: https://your-gitlab.com
  clientID: rhtas-client
```

3. Include templates in your pipeline:
```yaml
include:
  - remote: 'https://raw.githubusercontent.com/ihsanmokhlisse/rhtas-gitlab-ci-templates/main/templates/artifact-signing.yml'
```

## Basic Configuration

```yaml
variables:
  RHTAS_URL: "https://rhtas.example.com"
  RHTAS_OIDC_ISSUER: "https://gitlab.example.com"

sign-artifact:
  extends: .rhtas-sign-artifact
  variables:
    ARTIFACT_PATH: "target/*.jar"
```

## Verification

1. Run a test pipeline
2. Check RHTAS logs
3. Verify signature creation

## Next Steps

- Review [Template Reference](template-reference.md)
- Check [Security Best Practices](security.md)
- Explore [Examples](examples.md)