# Getting Started with RHTAS GitLab CI Templates

## Prerequisites

- GitLab CI/CD environment
- RHTAS instance (version 1.1.1+)
- OpenShift cluster (4.13+)
- GitLab OIDC provider configured in RHTAS

## Quick Start

1. Add template to your `.gitlab-ci.yml`:
```yaml
include:
  - remote: 'https://raw.githubusercontent.com/ihsanmokhlisse/rhtas-gitlab-ci-templates/main/templates/artifact-signing.yml'
```

2. Configure RHTAS variables:
```yaml
variables:
  RHTAS_URL: "https://your-rhtas-instance.com"
  RHTAS_OIDC_ISSUER: "https://your-gitlab.com"
```

3. Use the template:
```yaml
sign-artifact:
  extends: .rhtas-sign-artifact
  variables:
    ARTIFACT_PATH: "target/*.jar"
```

## Installation

1. RHTAS Setup
   - Install RHTAS operator via OLM
   - Configure OIDC integration
   - Set up required roles and permissions

2. GitLab Configuration
   - Enable OIDC token generation
   - Configure project CI/CD variables
   - Set up required permissions

## Next Steps

- Review the [Template Reference](template-reference.md)
- Check [Security Best Practices](security.md)
- See [Examples](examples.md)