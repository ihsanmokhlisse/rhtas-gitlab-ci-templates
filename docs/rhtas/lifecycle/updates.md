# RHTAS Update Procedures

## Overview

This guide details the update process for the Red Hat Trusted Artifact Signer (RHTAS) Operator and its components.

## Version Information

Current stable version (1.1.1) includes:

| Component | Version | Update Strategy |
|-----------|---------|-----------------|
| Fulcio | 1.6.5 | Automatic with operator |
| Rekor | 1.3.7 | Automatic with operator |
| Trillian | 1.7.0 | Automatic with operator |
| CT Log | 1.3.0 | Automatic with operator |
| TSA | 1.2.3 | Automatic with operator |

## Update Methods

### 1. Automatic Updates (Default)

The operator is configured for automatic updates when installed through OperatorHub:
- Update Channel: stable
- Approval Strategy: Automatic
- Installation Mode: All namespaces

### 2. Manual Update Process

For controlled environments:

```bash
# Check current subscription
oc get subscription securesign-operator -n openshift-operators -o yaml

# Set manual approval
oc patch subscription securesign-operator \
  -n openshift-operators \
  --type=merge \
  -p '{"spec":{"installPlanApproval":"Manual"}}'

# Review and approve updates
oc get installplan -n openshift-operators
oc patch installplan <install-plan-name> -n openshift-operators \
  --type=merge -p '{"spec":{"approved":true}}'
```

## Update Workflow

### 1. Pre-update Checks

```bash
# Verify current version
oc get csv -n trusted-artifact-signer

# Check operator health
oc get securesign -n trusted-artifact-signer

# Verify no pending operations
oc get pods -n trusted-artifact-signer
```

### 2. Backup Procedure

```bash
# Backup custom resources
oc get securesign -n trusted-artifact-signer -o yaml > securesign-backup.yaml

# Backup MariaDB data
oc exec -n trusted-artifact-signer \
  $(oc get pod -l app=mariadb -o jsonpath='{.items[0].metadata.name}') \
  -- mysqldump -u root -p${MYSQL_ROOT_PASSWORD} --all-databases > mariadb-backup.sql
```

### 3. Update Process

For automatic updates:
- Monitor operator status
- Check component versions
- Verify pod health

For manual updates:
```bash
# Review install plans
oc get installplan -n openshift-operators

# Approve update
oc patch installplan <install-plan-name> -n openshift-operators \
  --type=merge -p '{"spec":{"approved":true}}'
```

### 4. Post-update Verification

```bash
# Verify operator version
oc get csv -n trusted-artifact-signer

# Check component versions
oc get securesign -n trusted-artifact-signer -o yaml | grep -i version

# Verify all pods are running
oc get pods -n trusted-artifact-signer
```

## Monitoring During Updates

### 1. Key Metrics

```bash
# CPU and Memory Usage
container_cpu_usage_seconds_total{namespace="trusted-artifact-signer"}
container_memory_usage_bytes{namespace="trusted-artifact-signer"}

# Pod Status
kube_pod_status_phase{namespace="trusted-artifact-signer"}

# Database Metrics
mysql_global_status_threads_connected
mysql_global_status_questions
```

### 2. Alert Rules

```yaml
groups:
- name: RHTASUpdateAlerts
  rules:
  - alert: RHTASUpdateStalled
    expr: |
      changes(kube_pod_status_phase{namespace="trusted-artifact-signer",phase="Running"}[1h]) == 0
      and on(pod) kube_pod_container_status_waiting == 1
    for: 15m
    labels:
      severity: warning
    annotations:
      summary: "RHTAS update appears stalled"
```

## Troubleshooting Updates

### Common Issues

1. **Stuck Updates**:
```bash
# Check operator conditions
oc get securesign -n trusted-artifact-signer -o yaml | grep -A10 conditions

# Check operator logs
oc logs -n openshift-operators \
  deployment/securesign-operator-controller-manager
```

2. **Database Migration Issues**:
```bash
# Check MariaDB logs
oc logs -n trusted-artifact-signer \
  deployment/mariadb
```

### Recovery Procedures

1. **Rollback Process**:
```bash
# Revert to previous version
oc patch securesign -n trusted-artifact-signer \
  --type=merge \
  -p '{"spec":{"version":"<previous-version>"}}'
```

2. **Database Recovery**:
```bash
# Restore from backup
oc exec -n trusted-artifact-signer \
  $(oc get pod -l app=mariadb -o jsonpath='{.items[0].metadata.name}') \
  -- mysql -u root -p${MYSQL_ROOT_PASSWORD} < mariadb-backup.sql
```

## Best Practices

1. **Before Updates**:
   - Schedule during maintenance windows
   - Ensure sufficient cluster resources
   - Create backups
   - Document current configuration

2. **During Updates**:
   - Monitor logs
   - Watch for approvals
   - Verify database migrations
   - Check component health

3. **After Updates**:
   - Verify all components
   - Test signing workflows
   - Update client tools
   - Document new configurations

## Support

For update issues, contact:
- Maintainer: Ihsan Mokhlisse <imokhlis@redhat.com>
- Red Hat Support Portal