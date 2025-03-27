# RHTAS GitLab CI Templates

This repository contains reusable GitLab CI templates for integrating Red Hat Trusted Artifact Signer (RHTAS) into your CI/CD pipelines. These templates provide standardized configurations for signing artifacts and containers using RHTAS.

## Overview

Red Hat Trusted Artifact Signer (RHTAS) is a solution for signing and verifying software artifacts and container images. These templates make it easy to integrate RHTAS signing capabilities into your GitLab CI/CD pipelines.

## Prerequisites

- GitLab CI/CD
- Access to an RHTAS instance (version 1.1.1+)
- OpenShift cluster running RHTAS (4.13+)
- GitLab OIDC provider configured in RHTAS

## Quick Start

1. Include the desired template in your `.gitlab-ci.yml`:

```yaml
include:
  - remote: 'https://raw.githubusercontent.com/ihsanmokhlisse/rhtas-gitlab-ci-templates/main/templates/artifact-signing.yml'
```

2. Configure the required variables:

```yaml
variables:
  RHTAS_URL: "https://your-rhtas-instance.com"
  RHTAS_OIDC_ISSUER: "https://your-gitlab.com"
```

3. Use the template in your pipeline:

```yaml
sign-artifact:
  extends: .rhtas-sign-artifact
  variables:
    ARTIFACT_PATH: "target/*.jar"
```

## Repository Structure

```
.
├── templates/
│   ├── artifact-signing.yml
│   ├── container-signing.yml
│   └── verification.yml
├── examples/
│   ├── maven-artifact/
│   └── container-image/
└── docs/
    ├── getting-started.md     # Setup and installation guide
    ├── template-reference.md  # Detailed template documentation
    ├── security.md           # Security best practices
    └── examples.md           # Example implementations
```

## Documentation

Our documentation is organized by major topics:

- [Getting Started Guide](docs/getting-started.md) - Installation and basic setup
- [Template Reference](docs/template-reference.md) - Detailed template documentation
- [Security Best Practices](docs/security.md) - Security guidelines and recommendations
- [Examples](docs/examples.md) - Real-world implementation examples

## Configuration

### Required Variables

- `RHTAS_URL`: URL of your RHTAS instance
- `RHTAS_OIDC_ISSUER`: OIDC issuer URL (typically your GitLab instance)

### Optional Variables

- `RHTAS_NAMESPACE`: RHTAS namespace (default: trusted-artifact-signer)
- `SIGN_TIMEOUT`: Signing operation timeout in seconds (default: 300)
- `VERIFICATION_POLICY`: Policy for signature verification (default: strict)

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## License

Apache License 2.0

## Support

For issues and questions, please open an issue in this repository.