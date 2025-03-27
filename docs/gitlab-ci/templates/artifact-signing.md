# Artifact Signing Template Guide

## Overview

This guide explains how to use the RHTAS GitLab CI template for signing generic artifacts (Maven packages, binaries, etc.) in your CI/CD pipelines.

## Template Location

```yaml
include:
  - remote: 'https://raw.githubusercontent.com/ihsanmokhlisse/rhtas-gitlab-ci-templates/main/templates/artifact-signing.yml'
```

## Prerequisites

1. RHTAS (Red Hat Trusted Artifact Signer) instance running and accessible
2. GitLab project with CI/CD pipelines enabled
3. Artifacts built and available in the pipeline
4. OIDC authentication set up between GitLab and RHTAS

## Template Parameters

| Parameter | Description | Required | Default |
|-----------|-------------|----------|---------|
| `RHTAS_URL` | URL to RHTAS Fulcio service | Yes | - |
| `REKOR_URL` | URL to RHTAS Rekor service | Yes | - |
| `ARTIFACT_PATH` | Path to artifact(s) to sign | Yes | - |
| `COSIGN_VERSION` | Version of cosign to use | No | `v2.1.1` |
| `OIDC_TOKEN_FILE` | Path to OIDC token file | No | `.oidc-token` |
| `SIGNATURE_PATH` | Directory to store signatures | No | `signatures` |
| `EXTRA_ANNOTATIONS` | Additional annotations to add to signature | No | - |
| `ARTIFACT_TYPE` | Type of artifact being signed | No | `generic` |
| `SHA_ALGORITHM` | SHA algorithm to use for digest | No | `sha256` |

## Basic Usage

```yaml
stages:
  - build
  - sign

build-artifact:
  stage: build
  script:
    - mvn clean package
  artifacts:
    paths:
      - target/*.jar

sign-artifact:
  stage: sign
  extends: .artifact-signing
  variables:
    RHTAS_URL: https://fulcio.rhtas.example.com
    REKOR_URL: https://rekor.rhtas.example.com
    ARTIFACT_PATH: target/*.jar
```

## Advanced Configuration

### Custom Annotations

```yaml
sign-artifact:
  stage: sign
  extends: .artifact-signing
  variables:
    RHTAS_URL: https://fulcio.rhtas.example.com
    REKOR_URL: https://rekor.rhtas.example.com
    ARTIFACT_PATH: target/*.jar
    EXTRA_ANNOTATIONS: |
      build-date=$(date -u +'%Y-%m-%dT%H:%M:%SZ')
      git-commit=$CI_COMMIT_SHA
      pipeline-id=$CI_PIPELINE_ID
```

### Multiple Artifact Types

```yaml
sign-artifacts:
  stage: sign
  parallel:
    matrix:
      - ARTIFACT_TYPE: [
          "maven",
          "rpm",
          "deb"
        ]
        ARTIFACT_PATH: [
          "target/*.jar",
          "dist/*.rpm",
          "dist/*.deb"
        ]
  extends: .artifact-signing
  variables:
    RHTAS_URL: https://fulcio.rhtas.example.com
    REKOR_URL: https://rekor.rhtas.example.com
```

## Template Implementation Details

### Authentication

The template uses OIDC authentication to authenticate with RHTAS. GitLab generates an OIDC token that includes claims about the pipeline, project, and user. RHTAS verifies this token against the configured OIDC provider.

```yaml
.artifact-signing:
  script:
    - echo "$CI_JOB_JWT_V2" > $OIDC_TOKEN_FILE
    - export COSIGN_EXPERIMENTAL=1
    - mkdir -p $SIGNATURE_PATH
    - for artifact in $ARTIFACT_PATH; do
    -   sha_digest=$(openssl dgst -$SHA_ALGORITHM $artifact | awk '{print $2}')
    -   cosign sign-blob --oidc-issuer $CI_SERVER_URL --identity-token $OIDC_TOKEN_FILE --output-signature $SIGNATURE_PATH/$(basename $artifact).sig $artifact
    - done
```

### Signature Verification

After signing, the template verifies that the signature was successfully created and can be validated:

```yaml
.artifact-signing:
  script:
    # ... signing steps ...
    - for artifact in $ARTIFACT_PATH; do
    -   cosign verify-blob --certificate-identity $CI_PROJECT_PATH --certificate-oidc-issuer $CI_SERVER_URL --signature $SIGNATURE_PATH/$(basename $artifact).sig $artifact
    - done
```

## Integrating with CI/CD Workflows

### Maven Projects

```yaml
stages:
  - build
  - test
  - sign
  - deploy

build:
  stage: build
  script:
    - mvn clean package
  artifacts:
    paths:
      - target/*.jar

test:
  stage: test
  script:
    - mvn test

sign-artifact:
  stage: sign
  extends: .artifact-signing
  variables:
    RHTAS_URL: https://fulcio.rhtas.example.com
    REKOR_URL: https://rekor.rhtas.example.com
    ARTIFACT_PATH: target/*.jar
    ARTIFACT_TYPE: maven
  artifacts:
    paths:
      - target/*.jar
      - signatures/

deploy:
  stage: deploy
  script:
    - cosign verify-blob --certificate-identity $CI_PROJECT_PATH --certificate-oidc-issuer $CI_SERVER_URL --signature signatures/*.sig target/*.jar
    - mvn deploy:deploy-file -DgroupId=com.example -DartifactId=myapp -Dversion=1.0.0 -Dfile=target/myapp-1.0.0.jar -Durl=$MAVEN_REPO_URL
```

### NPM Projects

```yaml
stages:
  - build
  - sign
  - publish

build:
  stage: build
  script:
    - npm ci
    - npm run build
    - npm pack
  artifacts:
    paths:
      - *.tgz

sign-package:
  stage: sign
  extends: .artifact-signing
  variables:
    RHTAS_URL: https://fulcio.rhtas.example.com
    REKOR_URL: https://rekor.rhtas.example.com
    ARTIFACT_PATH: *.tgz
    ARTIFACT_TYPE: npm
  artifacts:
    paths:
      - *.tgz
      - signatures/

publish:
  stage: publish
  script:
    - cosign verify-blob --certificate-identity $CI_PROJECT_PATH --certificate-oidc-issuer $CI_SERVER_URL --signature signatures/*.sig *.tgz
    - npm publish *.tgz
```

## Security Considerations

1. **Credentials Handling**: Use GitLab CI/CD variables for sensitive information
2. **OIDC Configuration**: Ensure proper OIDC claim verification in RHTAS
3. **Signature Verification**: Always verify signatures before deploying artifacts
4. **Artifact Storage**: Ensure artifacts and signatures are properly secured

## Troubleshooting

### Common Issues

1. **Authentication failures**:
   - Verify OIDC provider configuration
   - Check GitLab project settings for JWT generation
   - Validate RHTAS policies

2. **Signing failures**:
   - Check RHTAS logs for errors
   - Verify connectivity to RHTAS services
   - Ensure artifacts exist and are accessible

3. **Verification failures**:
   - Check signature file matches the artifact
   - Verify RHTAS certificates
   - Check for artifact modifications

### Debugging Steps

```yaml
sign-artifact:
  stage: sign
  extends: .artifact-signing
  variables:
    RHTAS_URL: https://fulcio.rhtas.example.com
    REKOR_URL: https://rekor.rhtas.example.com
    ARTIFACT_PATH: target/*.jar
  before_script:
    - export COSIGN_DEBUG=1
    - curl -s $RHTAS_URL/api/v1/rootCert # Test connectivity
    - curl -s $REKOR_URL/api/v1/log/info # Test connectivity
    - find $ARTIFACT_PATH -type f | wc -l  # Verify artifacts exist
```

## Example Complete Pipeline

```yaml
include:
  - remote: 'https://raw.githubusercontent.com/ihsanmokhlisse/rhtas-gitlab-ci-templates/main/templates/artifact-signing.yml'

stages:
  - build
  - test
  - sign
  - deploy

variables:
  MAVEN_OPTS: "-Dmaven.repo.local=.m2/repository"

cache:
  paths:
    - .m2/repository

build:
  stage: build
  image: maven:3.8-openjdk-11
  script:
    - mvn clean package
  artifacts:
    paths:
      - target/*.jar

test:
  stage: test
  image: maven:3.8-openjdk-11
  script:
    - mvn test

sign-artifact:
  stage: sign
  extends: .artifact-signing
  variables:
    RHTAS_URL: https://fulcio.rhtas.example.com
    REKOR_URL: https://rekor.rhtas.example.com
    ARTIFACT_PATH: target/*.jar
    ARTIFACT_TYPE: maven
    EXTRA_ANNOTATIONS: |
      build-date=$(date -u +'%Y-%m-%dT%H:%M:%SZ')
      git-commit=$CI_COMMIT_SHA
      pipeline-id=$CI_PIPELINE_ID
  artifacts:
    paths:
      - target/*.jar
      - signatures/

deploy:
  stage: deploy
  image: maven:3.8-openjdk-11
  script:
    - cosign verify-blob --certificate-identity $CI_PROJECT_PATH --certificate-oidc-issuer $CI_SERVER_URL --signature signatures/*.sig target/*.jar
    - mvn deploy -DskipTests
  only:
    - main
```

## Support and Resources

- [RHTAS Documentation](https://docs.rhtas.example.com)
- [GitLab CI/CD Documentation](https://docs.gitlab.com/ee/ci/)
- [Cosign Documentation](https://github.com/sigstore/cosign)
- Maintainer Contact: Ihsan Mokhlisse <imokhlis@redhat.com>