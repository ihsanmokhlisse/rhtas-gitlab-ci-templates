# Getting Started with RHTAS GitLab CI Templates

This guide will help you set up and start using the RHTAS GitLab CI templates in your projects.

## Prerequisites

- GitLab CI/CD environment
- Access to an RHTAS instance (v1.1.1+)
- OpenShift cluster (4.13+) running RHTAS
- GitLab OIDC provider configured in RHTAS

## Installation

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
  claims:
    - project_path
    - namespace_path
```

3. Create signing policies in RHTAS:

```yaml
apiVersion: trusted.signing/v1alpha1
kind: SigningPolicy
metadata:
  name: gitlab-signing-policy
spec:
  oidcProvider: gitlab-provider
  conditions:
    - claim: project_path
      values: [your-group/your-project]
```

## Template Integration

1. Include the desired template in your `.gitlab-ci.yml`:

```yaml
include:
  - remote: 'https://raw.githubusercontent.com/ihsanmokhlisse/rhtas-gitlab-ci-templates/main/templates/artifact-signing.yml'
```

2. Configure required variables:

```yaml
variables:
  RHTAS_URL: "https://rhtas.example.com"
  RHTAS_OIDC_ISSUER: "https://gitlab.example.com"
```

3. Use the template in your pipeline:

```yaml
sign-artifact:
  extends: .rhtas-sign-artifact
  variables:
    ARTIFACT_PATH: "target/*.jar"
```

## Verification

To verify your setup:

1. Run a test pipeline
2. Check RHTAS logs for successful authentication
3. Verify signature creation
4. Test signature verification

## Next Steps

- Review the [Template Reference](template-reference.md) for detailed configuration options
- Check [Security Best Practices](security.md) for secure implementation
- Explore [Examples](examples.md) for real-world implementations