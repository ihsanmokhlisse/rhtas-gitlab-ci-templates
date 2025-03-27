# RHTAS Template Reference

## Available Templates

### Artifact Signing (.rhtas-sign-artifact)

```yaml
variables:
  RHTAS_URL: ""            # Required: RHTAS instance URL
  RHTAS_OIDC_ISSUER: ""    # Required: GitLab instance URL
  RHTAS_NAMESPACE: "trusted-artifact-signer"  # Optional
  SIGN_TIMEOUT: "300"      # Optional: Timeout in seconds
  ARTIFACT_PATH: ""        # Required: Path to artifact(s)
```

### Container Signing (.rhtas-sign-container)

```yaml
variables:
  RHTAS_URL: ""            # Required: RHTAS instance URL
  RHTAS_OIDC_ISSUER: ""    # Required: GitLab instance URL
  RHTAS_NAMESPACE: "trusted-artifact-signer"  # Optional
  SIGN_TIMEOUT: "300"      # Optional: Timeout in seconds
  CONTAINER_IMAGE: ""      # Required: Container image reference
```

### Verification (.rhtas-verify)

```yaml
variables:
  RHTAS_URL: ""            # Required: RHTAS instance URL
  VERIFICATION_POLICY: "strict"  # Optional: Verification policy
  TARGET_PATH: ""          # Required: Path/URL to verify
```

## Template Behaviors

- Templates run only on tagged commits by default
- Verification template runs on merge requests
- All templates use UBI minimal as base image
- Automatic cosign installation included