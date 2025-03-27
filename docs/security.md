# Security Best Practices

## OIDC Configuration

1. Token Security
   - Use short-lived tokens
   - Configure appropriate scopes
   - Enable token revocation

2. Access Control
   - Implement least privilege principle
   - Use protected branches
   - Configure merge request approvals

## Signing Process

1. Artifact Protection
   - Use immutable tags
   - Enable signature verification
   - Implement policy enforcement

2. Key Management
   - Use RHTAS key management
   - Regular key rotation
   - Secure key storage

## CI/CD Security

1. Pipeline Security
   - Protected variables
   - Secure runners
   - Environment isolation

2. Compliance
   - Audit logging
   - Policy enforcement
   - Compliance reporting