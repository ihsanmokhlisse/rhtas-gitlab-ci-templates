# SecureSign Instance Setup Guide

## Overview

This guide details the setup and configuration of a SecureSign instance in RHTAS.

## Prerequisites

- RHTAS Operator installed and running
- OpenShift cluster (4.13+)
- Cluster administrator access
- Storage class available for persistent volumes

## Instance Creation

### Basic Setup

```yaml
apiVersion: v1
kind: SecureSign
metadata:
  name: securesign
  namespace: trusted-artifact-signer
spec:
  version: "1.1.1"
  database:
    type: mariadb
  fulcio:
    enabled: true
  rekor:
    enabled: true
```

### Advanced Configuration

```yaml
apiVersion: v1
kind: SecureSign
metadata:
  name: securesign
  namespace: trusted-artifact-signer
spec:
  version: "1.1.1"
  database:
    type: mariadb
    external: false
    storage:
      class: managed-premium
      size: 10Gi
  fulcio:
    enabled: true
    replicas: 2
    resources:
      requests:
        memory: "512Mi"
        cpu: "250m"
      limits:
        memory: "1Gi"
        cpu: "500m"
  rekor:
    enabled: true
    replicas: 2
    resources:
      requests:
        memory: "512Mi"
        cpu: "250m"
      limits:
        memory: "1Gi"
        cpu: "500m"
  trillian:
    enabled: true
    replicas: 2
```

## Component Configuration

### 1. Database Setup

#### Internal MariaDB
```yaml
spec:
  database:
    type: mariadb
    storage:
      class: managed-premium
      size: 10Gi
    backup:
      enabled: true
      schedule: "0 2 * * *"
      retention: 7
```

#### External Database
```yaml
spec:
  database:
    type: mariadb
    external: true
    host: "external-db.example.com"
    port: 3306
    credentials:
      secretName: "external-db-credentials"
```

### 2. Fulcio Configuration

```yaml
spec:
  fulcio:
    enabled: true
    config:
      oidc:
        issuer: "https://gitlab.example.com"
        clientID: "rhtas-client"
      tls:
        secretName: "fulcio-tls"
    logging:
      level: "info"
```

### 3. Rekor Configuration

```yaml
spec:
  rekor:
    enabled: true
    storage:
      class: managed-premium
      size: 50Gi
    config:
      logIndex:
        treeID: 0
      attestation:
        storage:
          type: filesystem
```

## Security Configuration

### 1. OIDC Provider Setup

```yaml
apiVersion: trusted.signing/v1alpha1
kind: OIDCProvider
metadata:
  name: gitlab-provider
spec:
  issuer: https://gitlab.example.com
  clientID: rhtas-client
  claims:
    - project_path
    - namespace_path
```

### 2. Signing Policies

```yaml
apiVersion: trusted.signing/v1alpha1
kind: SigningPolicy
metadata:
  name: gitlab-signing-policy
spec:
  oidcProvider: gitlab-provider
  conditions:
    - claim: project_path
      values: ["group/project"]
```

## Monitoring Setup

### 1. Prometheus Integration

```yaml
spec:
  monitoring:
    enabled: true
    serviceMonitor: true
    grafanaDashboards: true
```

### 2. Logging Configuration

```yaml
spec:
  logging:
    level: "info"
    format: "json"
    output: "file"
```

## Verification Steps

1. Check instance status:
```bash
oc get securesign -n trusted-artifact-signer
```

2. Verify component pods:
```bash
oc get pods -n trusted-artifact-signer
```

3. Test connectivity:
```bash
# Test Fulcio
curl -k https://fulcio-server-trusted-artifact-signer.${APPS_DOMAIN}/api/v1/rootCert

# Test Rekor
curl -k https://rekor-server-trusted-artifact-signer.${APPS_DOMAIN}/api/v1/log
```

## Troubleshooting

### Common Issues

1. **Database Connection**:
```bash
# Check database logs
oc logs -n trusted-artifact-signer deployment/mariadb

# Verify credentials
oc get secret -n trusted-artifact-signer mariadb-credentials
```

2. **Component Health**:
```bash
# Check component status
oc describe securesign -n trusted-artifact-signer

# View component logs
oc logs -n trusted-artifact-signer deployment/fulcio-server
oc logs -n trusted-artifact-signer deployment/rekor-server
```

3. **Storage Issues**:
```bash
# Check PVCs
oc get pvc -n trusted-artifact-signer

# Verify storage class
oc get storageclass
```

## Best Practices

1. **High Availability**:
   - Deploy multiple replicas
   - Use external database for production
   - Configure proper resource limits

2. **Security**:
   - Enable TLS
   - Configure OIDC properly
   - Implement signing policies

3. **Monitoring**:
   - Enable Prometheus metrics
   - Set up alerts
   - Configure proper logging

## Next Steps

1. [Configure Access Control](../admin/access-control.md)
2. [Setup Monitoring](../admin/monitoring.md)
3. [Configure Backup](../lifecycle/backup-recovery.md)

## Support

For setup issues, contact:
- Maintainer: Ihsan Mokhlisse <imokhlis@redhat.com>
- Red Hat Support Portal