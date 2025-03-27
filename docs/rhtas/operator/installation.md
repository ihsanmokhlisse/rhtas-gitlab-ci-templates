# RHTAS Operator Installation Guide

## Overview

This guide covers the installation process for the Red Hat Trusted Artifact Signer (RHTAS) Operator on OpenShift.

## Prerequisites

- OpenShift cluster (version 4.13+)
- Cluster administrator access
- Sufficient cluster resources
- Access to Red Hat registry

## Installation Methods

### 1. OperatorHub Installation (Recommended)

1. Log in to OpenShift Web Console
2. Navigate to **Operators** → **OperatorHub**
3. Search for "RHTAS" or "Trusted Artifact Signer"
4. Click **Install**
5. Configure installation parameters:
   - Update Channel: stable
   - Installation Mode: All namespaces
   - Approval Strategy: Automatic (recommended)

### 2. CLI Installation

```bash
# Create namespace
oc create namespace trusted-artifact-signer

# Apply operator group
cat <<EOF | oc apply -f -
apiVersion: operators.coreos.com/v1
kind: OperatorGroup
metadata:
  name: rhtas-operator-group
  namespace: trusted-artifact-signer
spec:
  targetNamespaces:
  - trusted-artifact-signer
EOF

# Subscribe to the operator
cat <<EOF | oc apply -f -
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: securesign-operator
  namespace: trusted-artifact-signer
spec:
  channel: stable
  name: securesign-operator
  source: redhat-operators
  sourceNamespace: openshift-marketplace
EOF
```

## Post-Installation Verification

1. Check operator status:
```bash
oc get csv -n trusted-artifact-signer
```

2. Verify operator pod:
```bash
oc get pods -n trusted-artifact-signer
```

3. Check operator logs:
```bash
oc logs -n trusted-artifact-signer \
  deployment/securesign-operator-controller-manager
```

## Configuration

### Basic Configuration

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

See [Configuration Guide](configuration.md) for detailed settings.

## Troubleshooting

### Common Installation Issues

1. **ImagePullBackOff**:
   - Verify registry access
   - Check pull secrets
   - Validate network connectivity

2. **Pending State**:
   - Check resource availability
   - Verify node scheduling
   - Review events

3. **CrashLoopBackOff**:
   - Check operator logs
   - Verify configuration
   - Review resource limits

## Next Steps

1. [Configure the Operator](configuration.md)
2. [Create SecureSign Instance](../securesign/setup.md)
3. [Setup User Access](../admin/user-management.md)

## Support

For installation issues, contact:
- Maintainer: Ihsan Mokhlisse <imokhlis@redhat.com>
- Red Hat Support Portal