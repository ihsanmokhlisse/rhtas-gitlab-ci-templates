# Template Reference

Comprehensive documentation for RHTAS GitLab CI templates.

## Available Templates

### artifact-signing.yml

```yaml
.rhtas-sign-artifact:
  variables:
    RHTAS_URL: ""
    RHTAS_OIDC_ISSUER: ""
    ARTIFACT_PATH: ""
    SIGN_TIMEOUT: "300"
```

**Variables:**
- `RHTAS_URL`: RHTAS instance URL (Required)
- `RHTAS_OIDC_ISSUER`: OIDC issuer URL (Required)
- `ARTIFACT_PATH`: Path to artifact(s) (Required)
- `SIGN_TIMEOUT`: Timeout in seconds (Optional)

### container-signing.yml

```yaml
.rhtas-sign-container:
  variables:
    RHTAS_URL: ""
    RHTAS_OIDC_ISSUER: ""
    IMAGE_REFERENCE: ""
```

**Variables:**
- `RHTAS_URL`: RHTAS instance URL (Required)
- `RHTAS_OIDC_ISSUER`: OIDC issuer URL (Required)
- `IMAGE_REFERENCE`: Container image reference (Required)

### verification.yml

```yaml
.rhtas-verify:
  variables:
    RHTAS_URL: ""
    VERIFICATION_POLICY: "strict"
    TARGET_PATH: ""
```

**Variables:**
- `RHTAS_URL`: RHTAS instance URL (Required)
- `VERIFICATION_POLICY`: Verification policy (Optional)
- `TARGET_PATH`: Path to verify (Required)

## Common Configuration

```yaml
variables:
  RHTAS_NAMESPACE: "trusted-artifact-signer"
  RHTAS_LOG_LEVEL: "info"
```

## Pipeline Configuration

```yaml
default:
  tags:
    - rhtas
  retry:
    max: 2
    when:
      - runner_system_failure
```