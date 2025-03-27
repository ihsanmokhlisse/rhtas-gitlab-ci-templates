# RHTAS Ansible Automation Guide

## Overview

This guide provides comprehensive documentation for automating RHTAS deployment and management using Ansible.

## Prerequisites

- Ansible 2.9+
- OpenShift Python client (`openshift` Python package)
- OpenShift CLI (`oc`)
- Access to OpenShift cluster
- RHTAS operator subscription

## Playbook Structure

```plaintext
rhtas-automation/
├── inventory/
│   ├── group_vars/
│   │   └── all.yml
│   └── hosts
├── roles/
│   ├── rhtas-operator/
│   ├── securesign-instance/
│   └── rhtas-config/
├── playbooks/
│   ├── install.yml
│   ├── configure.yml
│   └── manage.yml
└── ansible.cfg
```

## Installation Playbook

### Operator Installation

```yaml
# playbooks/install.yml
---
- name: Install RHTAS Operator
  hosts: localhost
  gather_facts: false
  tasks:
    - name: Create namespace
      k8s:
        state: present
        definition:
          apiVersion: v1
          kind: Namespace
          metadata:
            name: trusted-artifact-signer

    - name: Create operator group
      k8s:
        state: present
        definition:
          apiVersion: operators.coreos.com/v1
          kind: OperatorGroup
          metadata:
            name: rhtas-operator
            namespace: trusted-artifact-signer
          spec:
            targetNamespaces:
              - trusted-artifact-signer

    - name: Create subscription
      k8s:
        state: present
        definition:
          apiVersion: operators.coreos.com/v1alpha1
          kind: Subscription
          metadata:
            name: rhtas-operator
            namespace: trusted-artifact-signer
          spec:
            channel: stable-v1
            name: rhtas-operator
            source: redhat-operators
            sourceNamespace: openshift-marketplace
```

### SecureSign Instance Deployment

```yaml
# roles/securesign-instance/tasks/main.yml
---
- name: Deploy SecureSign instance
  k8s:
    state: present
    definition:
      apiVersion: v1
      kind: SecureSign
      metadata:
        name: "{{ securesign_name }}"
        namespace: "{{ namespace }}"
      spec:
        version: "{{ rhtas_version }}"
        database:
          type: "{{ db_type }}"
          storage:
            class: "{{ storage_class }}"
            size: "{{ db_storage_size }}"
        fulcio:
          enabled: true
          replicas: "{{ fulcio_replicas }}"
        rekor:
          enabled: true
          replicas: "{{ rekor_replicas }}"

- name: Wait for SecureSign deployment
  k8s_info:
    api_version: v1
    kind: SecureSign
    name: "{{ securesign_name }}"
    namespace: "{{ namespace }}"
  register: securesign_status
  until: securesign_status.resources[0].status.phase == "Ready"
  retries: 30
  delay: 10
```

## Configuration Management

### OIDC Provider Setup

```yaml
# roles/rhtas-config/tasks/oidc.yml
---
- name: Configure OIDC provider
  k8s:
    state: present
    definition:
      apiVersion: trusted.signing/v1alpha1
      kind: OIDCProvider
      metadata:
        name: "{{ oidc_provider_name }}"
        namespace: "{{ namespace }}"
      spec:
        issuer: "{{ oidc_issuer }}"
        clientID: "{{ oidc_client_id }}"
        clientSecret:
          name: "{{ oidc_secret_name }}"
          key: client-secret
```

### Signing Policy Configuration

```yaml
# roles/rhtas-config/tasks/policies.yml
---
- name: Configure signing policies
  k8s:
    state: present
    definition:
      apiVersion: trusted.signing/v1alpha1
      kind: SigningPolicy
      metadata:
        name: "{{ item.name }}"
        namespace: "{{ namespace }}"
      spec:
        oidcProvider: "{{ item.oidc_provider }}"
        conditions:
          - claim: "{{ item.claim }}"
            values: "{{ item.values }}"
  loop: "{{ signing_policies }}"
```

## Maintenance Tasks

### Backup Configuration

```yaml
# roles/rhtas-config/tasks/backup.yml
---
- name: Configure backup
  k8s:
    state: present
    definition:
      apiVersion: v1
      kind: SecureSign
      metadata:
        name: "{{ securesign_name }}"
        namespace: "{{ namespace }}"
      spec:
        backup:
          enabled: true
          schedule: "{{ backup_schedule }}"
          retention: "{{ backup_retention }}"
          storage:
            class: "{{ backup_storage_class }}"
            size: "{{ backup_storage_size }}"
```

### Resource Management

```yaml
# roles/rhtas-config/tasks/resources.yml
---
- name: Update resource configuration
  k8s:
    state: present
    definition:
      apiVersion: v1
      kind: SecureSign
      metadata:
        name: "{{ securesign_name }}"
        namespace: "{{ namespace }}"
      spec:
        fulcio:
          resources:
            requests:
              cpu: "{{ fulcio_cpu_request }}"
              memory: "{{ fulcio_memory_request }}"
            limits:
              cpu: "{{ fulcio_cpu_limit }}"
              memory: "{{ fulcio_memory_limit }}"
        rekor:
          resources:
            requests:
              cpu: "{{ rekor_cpu_request }}"
              memory: "{{ rekor_memory_request }}"
            limits:
              cpu: "{{ rekor_cpu_limit }}"
              memory: "{{ rekor_memory_limit }}"
```

## Variables Configuration

### Group Variables

```yaml
# inventory/group_vars/all.yml
---
# General settings
namespace: trusted-artifact-signer
rhtas_version: "1.1.1"
securesign_name: securesign

# Database settings
db_type: mariadb
storage_class: managed-premium
db_storage_size: 10Gi

# Component settings
fulcio_replicas: 2
rekor_replicas: 2

# OIDC settings
oidc_provider_name: gitlab-provider
oidc_issuer: https://gitlab.example.com
oidc_client_id: rhtas-client
oidc_secret_name: gitlab-oidc-secret

# Resource settings
fulcio_cpu_request: "250m"
fulcio_memory_request: "512Mi"
fulcio_cpu_limit: "500m"
fulcio_memory_limit: "1Gi"

rekor_cpu_request: "250m"
rekor_memory_request: "512Mi"
rekor_cpu_limit: "500m"
rekor_memory_limit: "1Gi"

# Backup settings
backup_schedule: "0 2 * * *"
backup_retention: 7
backup_storage_class: managed-premium
backup_storage_size: 100Gi
```

## Usage Examples

### Basic Installation

```bash
# Install RHTAS operator
ansible-playbook playbooks/install.yml

# Deploy and configure SecureSign instance
ansible-playbook playbooks/configure.yml
```

### Configuration Updates

```bash
# Update resource configuration
ansible-playbook playbooks/manage.yml --tags resources

# Configure backup
ansible-playbook playbooks/manage.yml --tags backup
```

## Troubleshooting

### Common Issues

1. **Operator Installation Fails**:
```bash
# Check operator status
oc get csv -n trusted-artifact-signer

# View operator logs
oc logs -n trusted-artifact-signer deployment/rhtas-operator
```

2. **SecureSign Deployment Issues**:
```bash
# Check SecureSign status
oc get securesign -n trusted-artifact-signer

# View component events
oc get events -n trusted-artifact-signer
```

### Playbook Debugging

```bash
# Enable verbose output
ansible-playbook playbooks/install.yml -vvv

# Check syntax
ansible-playbook playbooks/install.yml --syntax-check
```

## Best Practices

1. **Version Control**:
   - Use Git for playbook management
   - Tag releases
   - Document changes

2. **Security**:
   - Use vault for sensitive data
   - Implement RBAC
   - Regular secret rotation

3. **Testing**:
   - Test in development first
   - Use molecule for role testing
   - Implement idempotency tests

## Support

For automation issues:
- Check playbook logs
- Review Ansible documentation
- Contact maintainer: Ihsan Mokhlisse <imokhlis@redhat.com>