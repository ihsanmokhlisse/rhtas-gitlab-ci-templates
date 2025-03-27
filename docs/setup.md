# RHTAS GitLab CI Setup Guide

## Prerequisites

1. RHTAS Instance
   - Ensure RHTAS 1.1.1+ is installed on your OpenShift cluster
   - Configure GitLab as an OIDC provider

2. GitLab Configuration
   - Enable OIDC token generation for CI jobs
   - Configure appropriate permissions

## Installation

1. Add the templates to your project:
   ```yaml
   include:
     - remote: 'https://raw.githubusercontent.com/ihsanmokhlisse/rhtas-gitlab-ci-templates/main/templates/artifact-signing.yml'
     - remote: 'https://raw.githubusercontent.com/ihsanmokhlisse/rhtas-gitlab-ci-templates/main/templates/container-signing.yml'
     - remote: 'https://raw.githubusercontent.com/ihsanmokhlisse/rhtas-gitlab-ci-templates/main/templates/verification.yml'
   ```

2. Configure RHTAS variables in your CI/CD settings or `.gitlab-ci.yml`

3. Create signing jobs by extending the templates

## Verification

Test your setup by:
1. Creating a test artifact/container
2. Running a signing job
3. Verifying the signature

## Next Steps

- Review the [Configuration Guide](configuration.md)
- Check [Troubleshooting Guide](troubleshooting.md) if you encounter issues