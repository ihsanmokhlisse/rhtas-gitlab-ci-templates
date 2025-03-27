# Red Hat Trusted Artifact Signer (RHTAS) Documentation

## Overview

This repository contains comprehensive documentation for Red Hat Trusted Artifact Signer (RHTAS), including operator guides, automation tools, and GitLab CI integration templates.

## Documentation Structure

### 1. RHTAS Core Documentation

#### Operator
- [Installation Guide](docs/rhtas/operator/installation.md)
- [Configuration Guide](docs/rhtas/operator/configuration.md)
- [Troubleshooting Guide](docs/rhtas/operator/troubleshooting.md)

#### SecureSign Instance
- [Instance Setup](docs/rhtas/securesign/setup.md)
- [Instance Configuration](docs/rhtas/securesign/configuration.md)
- [Troubleshooting Guide](docs/rhtas/securesign/troubleshooting.md)

#### Management and Administration
- [Administrative Tasks](docs/rhtas/admin/tasks.md)
- [User Management](docs/rhtas/admin/user-management.md)
- [Access Control](docs/rhtas/admin/access-control.md)
- [Monitoring and Logging](docs/rhtas/admin/monitoring.md)
- [Troubleshooting Guide](docs/rhtas/admin/troubleshooting.md)

#### Lifecycle Management
- [Update Procedures](docs/rhtas/lifecycle/updates.md)
- [Backup and Recovery](docs/rhtas/lifecycle/backup-recovery.md)
- [Version Compatibility](docs/rhtas/lifecycle/compatibility.md)
- [Troubleshooting Guide](docs/rhtas/lifecycle/troubleshooting.md)

### 2. Automation with Ansible

- [Ansible Integration](docs/automation/ansible/integration.md)
- [Playbook Reference](docs/automation/ansible/playbooks.md)
- [Role Documentation](docs/automation/ansible/roles.md)
- [Variables Reference](docs/automation/ansible/variables.md)
- [Troubleshooting Guide](docs/automation/ansible/troubleshooting.md)

### 3. GitLab CI Integration

#### Templates
- [Template Reference](docs/gitlab-ci/templates/reference.md)
- [Configuration Guide](docs/gitlab-ci/templates/configuration.md)
- [Security Best Practices](docs/gitlab-ci/templates/security.md)

#### Pipeline Integration
- [Pipeline Setup](docs/gitlab-ci/pipeline/setup.md)
- [OIDC Configuration](docs/gitlab-ci/pipeline/oidc.md)
- [Artifact Signing](docs/gitlab-ci/pipeline/signing.md)
- [Container Signing](docs/gitlab-ci/pipeline/containers.md)
- [Troubleshooting Guide](docs/gitlab-ci/pipeline/troubleshooting.md)

## Quick Start Guides

1. [RHTAS Operator Installation](docs/quickstart/operator-install.md)
2. [SecureSign Instance Setup](docs/quickstart/securesign-setup.md)
3. [GitLab CI Integration](docs/quickstart/gitlab-integration.md)
4. [Ansible Automation](docs/quickstart/ansible-automation.md)

## Detailed Guides

### RHTAS Implementation
- [Architecture Overview](docs/guides/rhtas/architecture.md)
- [Security Considerations](docs/guides/rhtas/security.md)
- [Performance Tuning](docs/guides/rhtas/performance.md)
- [High Availability](docs/guides/rhtas/high-availability.md)

### Automation Implementation
- [CI/CD Integration](docs/guides/automation/cicd.md)
- [Custom Playbook Development](docs/guides/automation/custom-playbooks.md)
- [Advanced Configurations](docs/guides/automation/advanced-config.md)

### GitLab CI Implementation
- [Advanced Pipeline Patterns](docs/guides/gitlab-ci/advanced-patterns.md)
- [Custom Template Development](docs/guides/gitlab-ci/custom-templates.md)
- [Security Hardening](docs/guides/gitlab-ci/security-hardening.md)

## Troubleshooting

- [Common Issues](docs/troubleshooting/common-issues.md)
- [Known Limitations](docs/troubleshooting/limitations.md)
- [FAQs](docs/troubleshooting/faq.md)

## Reference

- [API Documentation](docs/reference/api.md)
- [CLI Reference](docs/reference/cli.md)
- [Configuration Reference](docs/reference/configuration.md)
- [Variable Reference](docs/reference/variables.md)

## Contributing

Please see our [Contributing Guide](CONTRIBUTING.md) for details on how to contribute to this documentation.

## Support

For issues or questions, contact:
- Maintainer: Ihsan Mokhlisse <imokhlis@redhat.com>

## License

This documentation is licensed under the Apache License 2.0 - see the [LICENSE](LICENSE) file for details.