# Security Best Practices

Security guidelines for RHTAS GitLab CI templates.

## OIDC Configuration

### Best Practices

1. Use short-lived tokens
2. Configure minimal required claims
3. Implement strict audience validation
4. Use HTTPS for all endpoints

### Claims Configuration

```yaml
spec:
  claims:
    - project_path
    - namespace_path
  required_claims:
    - project_path
```

## Access Control

### Signing Policies

```yaml
apiVersion: trusted.signing/v1alpha1
kind: SigningPolicy
metadata:
  name: restricted-policy
spec:
  conditions:
    - claim: project_path
      values: ["group/project"]
```

## Pipeline Security

### Protected Variables

```yaml
variables:
  RHTAS_TOKEN:
    value: "secret"
    protected: true
    masked: true
```

### Runner Configuration

```yaml
runner:
  tags:
    - rhtas-secure
  resource_limits:
    memory: 512Mi
    cpu: 0.5
```

## Artifact Security

### Secure Storage

```yaml
artifacts:
  reports:
    signatures: signatures/
  expire_in: 1 week
  encrypted: true
```

## Monitoring

### Logging

```yaml
variables:
  RHTAS_LOG_LEVEL: debug
  RHTAS_AUDIT: "true"
```

### Alerts

```yaml
on_failure:
  script:
    - notify_security_team
```