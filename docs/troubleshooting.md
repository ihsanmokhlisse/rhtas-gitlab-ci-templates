# RHTAS GitLab CI Troubleshooting Guide

## Common Issues

### Authentication Failures

1. OIDC Token Issues
   - Verify GitLab OIDC configuration
   - Check token permissions
   - Validate RHTAS OIDC settings

2. Connection Problems
   - Verify RHTAS URL accessibility
   - Check network policies
   - Validate TLS certificates

3. Signing Failures
   - Verify artifact/container accessibility
   - Check storage permissions
   - Validate signing key access

## Debugging Steps

1. Enable debug logging:
   ```yaml
   variables:
     COSIGN_DEBUG: 1
   ```

2. Verify RHTAS connectivity:
   ```bash
   curl -v "${RHTAS_URL}/api/fulcio/health"
   ```

3. Validate OIDC token:
   ```bash
   echo $CI_JOB_JWT_V2 | jwt decode
   ```

## Support

If issues persist:
1. Check RHTAS logs
2. Review GitLab job logs
3. Open an issue with details