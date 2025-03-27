# Template Reference

Detailed documentation for all available RHTAS GitLab CI templates.

## Available Templates

### artifact-signing.yml

Template for signing generic artifacts.

```yaml
.rhtas-sign-artifact:
  variables:
    RHTAS_URL: ""
    RHTAS_OIDC_ISSUER: ""
    ARTIFACT_PATH: ""
    SIGN_TIMEOUT: "300"
  script:
    - rhtas sign artifact $ARTIFACT_PATH
```

#### Variables

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| RHTAS_URL | Yes | - | RHTAS instance URL |
| RHTAS_OIDC_ISSUER | Yes | - | OIDC issuer URL |
| ARTIFACT_PATH | Yes | - | Path to artifact(s) |
| SIGN_TIMEOUT | No | 300 | Timeout in seconds |

### container-signing.yml

Template for signing container images.

```yaml
.rhtas-sign-container:
  variables:
    RHTAS_URL: ""
    RHTAS_OIDC_ISSUER: ""
    IMAGE_REFERENCE: ""
    SIGN_TIMEOUT: "300"
  script:
    - rhtas sign container $IMAGE_REFERENCE
```

#### Variables

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| RHTAS_URL | Yes | - | RHTAS instance URL |
| RHTAS_OIDC_ISSUER | Yes | - | OIDC issuer URL |
| IMAGE_REFERENCE | Yes | - | Container image reference |
| SIGN_TIMEOUT | No | 300 | Timeout in seconds |

### verification.yml

Template for verifying signatures.

```yaml
.rhtas-verify:
  variables:
    RHTAS_URL: ""
    VERIFICATION_POLICY: "strict"
    TARGET_PATH: ""
  script:
    - rhtas verify $TARGET_PATH
```

#### Variables

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| RHTAS_URL | Yes | - | RHTAS instance URL |
| VERIFICATION_POLICY | No | strict | Verification policy |
| TARGET_PATH | Yes | - | Path to verify |

## Common Configuration

### Environment Variables

```yaml
variables:
  RHTAS_NAMESPACE: "trusted-artifact-signer"
  RHTAS_LOG_LEVEL: "info"
```

### Pipeline Configuration

```yaml
default:
  tags:
    - rhtas
  retry:
    max: 2
    when:
      - runner_system_failure
      - stuck_or_timeout_failure
```

## Advanced Usage

### Custom Verification Policies

```yaml
variables:
  VERIFICATION_POLICY: |
    {"type": "strict",
     "required_signatures": 2,
     "trusted_identities": ["signer1", "signer2"]}
```

### Parallel Signing

```yaml
sign-artifacts:
  parallel:
    matrix:
      - ARTIFACT: ["lib/*.jar", "bin/*.exe"]
  extends: .rhtas-sign-artifact
  variables:
    ARTIFACT_PATH: $ARTIFACT
```