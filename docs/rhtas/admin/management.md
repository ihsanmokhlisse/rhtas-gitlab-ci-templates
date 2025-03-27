# RHTAS Management and Administration Guide

## Overview

This guide covers the management and administration tasks for Red Hat Trusted Artifact Signer (RHTAS).

## Access Control Management

### OIDC Configuration

1. **Adding OIDC Providers**:
```yaml
apiVersion: trusted.signing/v1alpha1
kind: OIDCProvider
metadata:
  name: gitlab-provider
spec:
  issuer: https://gitlab.example.com
  clientID: rhtas-client
  clientSecret:
    name: gitlab-oidc-secret
    key: client-secret
```

2. **Managing OIDC Claims**:
```yaml
spec:
  claims:
    required:
      - project_path
      - namespace_id
    optional:
      - user_email
      - group_path
```

### Role-Based Access Control

1. **Creating Role Bindings**:
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: rhtas-admin
  namespace: trusted-artifact-signer
subjects:
- kind: User
  name: admin@example.com
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: rhtas-admin
  apiGroup: rbac.authorization.k8s.io
```

2. **Custom Roles**:
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: rhtas-viewer
  namespace: trusted-artifact-signer
rules:
- apiGroups: ["trusted.signing"]
  resources: ["securesigns", "signingpolicies"]
  verbs: ["get", "list", "watch"]
```

## Resource Management

### Scaling Components

1. **Horizontal Scaling**:
```yaml
spec:
  fulcio:
    replicas: 3
  rekor:
    replicas: 3
  trillian:
    replicas: 2
```

2. **Resource Allocation**:
```yaml
spec:
  fulcio:
    resources:
      requests:
        cpu: "500m"
        memory: "1Gi"
      limits:
        cpu: "1"
        memory: "2Gi"
```

### Storage Management

1. **PVC Configuration**:
```yaml
spec:
  database:
    storage:
      class: managed-premium
      size: 20Gi
  rekor:
    storage:
      class: managed-premium
      size: 50Gi
```

2. **Backup Configuration**:
```yaml
spec:
  backup:
    enabled: true
    schedule: "0 2 * * *"
    retention: 7
    storage:
      class: managed-premium
      size: 100Gi
```

## Monitoring and Alerting

### Prometheus Integration

1. **Enable Monitoring**:
```yaml
spec:
  monitoring:
    enabled: true
    serviceMonitor: true
```

2. **Alert Rules**:
```yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: rhtas-alerts
  namespace: trusted-artifact-signer
spec:
  groups:
  - name: rhtas
    rules:
    - alert: RHTASHighErrorRate
      expr: rate(rhtas_error_total[5m]) > 0.1
      for: 5m
      labels:
        severity: warning
```

### Logging Configuration

1. **Component Logging**:
```yaml
spec:
  logging:
    level: info
    format: json
    output: file
```

2. **Log Aggregation**:
```yaml
spec:
  logging:
    forwarding:
      enabled: true
      type: elasticsearch
      endpoint: http://elasticsearch:9200
```

## Performance Tuning

### Database Optimization

1. **Connection Pool**:
```yaml
spec:
  database:
    pool:
      maxConnections: 100
      minConnections: 10
      maxIdleTime: "30m"
```

2. **Query Cache**:
```yaml
spec:
  database:
    cache:
      enabled: true
      size: "256MB"
```

### Component Optimization

1. **Fulcio Settings**:
```yaml
spec:
  fulcio:
    config:
      maxConcurrentRequests: 100
      requestTimeout: "30s"
```

2. **Rekor Settings**:
```yaml
spec:
  rekor:
    config:
      maxEntrySize: "10MB"
      batchSize: 100
```

## Maintenance Procedures

### Backup and Recovery

1. **Manual Backup**:
```bash
# Create backup
oc exec -n trusted-artifact-signer deployment/rhtas-backup -- backup create

# List backups
oc exec -n trusted-artifact-signer deployment/rhtas-backup -- backup list
```

2. **Restore Procedure**:
```bash
# Restore from backup
oc exec -n trusted-artifact-signer deployment/rhtas-backup -- restore --backup-name backup-20250327
```

### Version Management

1. **Upgrade Process**:
```yaml
spec:
  version: "1.1.2"
  upgradeStrategy:
    type: RollingUpdate
    maxUnavailable: 1
```

2. **Rollback Procedure**:
```yaml
spec:
  version: "1.1.1"
  rollback:
    enabled: true
    timeout: "1h"
```

## Troubleshooting

### Health Checks

1. **Component Status**:
```bash
# Check overall status
oc get securesign -n trusted-artifact-signer

# Check component health
oc describe securesign -n trusted-artifact-signer
```

2. **Log Analysis**:
```bash
# View component logs
oc logs -n trusted-artifact-signer deployment/fulcio-server
oc logs -n trusted-artifact-signer deployment/rekor-server
```

### Common Issues

1. **Database Connectivity**:
```bash
# Check database connection
oc exec -n trusted-artifact-signer deployment/mariadb -- mysql -u root -p -e "SHOW PROCESSLIST"
```

2. **Certificate Issues**:
```bash
# Check certificate status
oc get secret -n trusted-artifact-signer
oc describe secret tls-secret -n trusted-artifact-signer
```

## Best Practices

1. **Security**:
   - Regular certificate rotation
   - Implement network policies
   - Enable audit logging

2. **Performance**:
   - Monitor resource usage
   - Implement proper scaling
   - Regular maintenance windows

3. **Backup**:
   - Regular backup testing
   - Off-site backup copies
   - Document recovery procedures

## Support and Escalation

For support:
1. Check documentation
2. Review logs and metrics
3. Contact Red Hat Support
4. Escalate to maintainer: Ihsan Mokhlisse <imokhlis@redhat.com>