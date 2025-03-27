# Examples

Real-world examples of RHTAS GitLab CI template implementations.

## Maven Artifact Signing

### Pipeline Configuration

```yaml
include:
  - remote: 'https://raw.githubusercontent.com/ihsanmokhlisse/rhtas-gitlab-ci-templates/main/templates/artifact-signing.yml'

variables:
  RHTAS_URL: "https://rhtas.example.com"
  RHTAS_OIDC_ISSUER: "https://gitlab.example.com"

build:
  stage: build
  script:
    - mvn clean package
  artifacts:
    paths:
      - target/*.jar

sign:
  stage: sign
  extends: .rhtas-sign-artifact
  variables:
    ARTIFACT_PATH: "target/*.jar"
  dependencies:
    - build

verify:
  stage: verify
  extends: .rhtas-verify
  variables:
    TARGET_PATH: "target/*.jar"
  dependencies:
    - sign
```

## Container Image Signing

### Pipeline Configuration

```yaml
include:
  - remote: 'https://raw.githubusercontent.com/ihsanmokhlisse/rhtas-gitlab-ci-templates/main/templates/container-signing.yml'

variables:
  RHTAS_URL: "https://rhtas.example.com"
  RHTAS_OIDC_ISSUER: "https://gitlab.example.com"
  IMAGE_NAME: "registry.example.com/app"
  IMAGE_TAG: "$CI_COMMIT_SHA"

build:
  stage: build
  script:
    - docker build -t $IMAGE_NAME:$IMAGE_TAG .
    - docker push $IMAGE_NAME:$IMAGE_TAG

sign:
  stage: sign
  extends: .rhtas-sign-container
  variables:
    IMAGE_REFERENCE: "$IMAGE_NAME:$IMAGE_TAG"
  dependencies:
    - build

verify:
  stage: verify
  extends: .rhtas-verify
  variables:
    TARGET_PATH: "$IMAGE_NAME:$IMAGE_TAG"
  dependencies:
    - sign
```

## Multi-Stage Pipeline

### Pipeline Configuration

```yaml
include:
  - remote: 'https://raw.githubusercontent.com/ihsanmokhlisse/rhtas-gitlab-ci-templates/main/templates/artifact-signing.yml'
  - remote: 'https://raw.githubusercontent.com/ihsanmokhlisse/rhtas-gitlab-ci-templates/main/templates/container-signing.yml'

variables:
  RHTAS_URL: "https://rhtas.example.com"
  RHTAS_OIDC_ISSUER: "https://gitlab.example.com"

stages:
  - build
  - test
  - sign
  - verify
  - deploy

build-app:
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
  extends: .rhtas-sign-artifact
  variables:
    ARTIFACT_PATH: "target/*.jar"
  dependencies:
    - build-app

build-image:
  stage: build
  script:
    - docker build -t $IMAGE:$TAG .
    - docker push $IMAGE:$TAG
  dependencies:
    - sign-artifact

sign-image:
  stage: sign
  extends: .rhtas-sign-container
  variables:
    IMAGE_REFERENCE: "$IMAGE:$TAG"
  dependencies:
    - build-image

verify-signatures:
  stage: verify
  parallel:
    matrix:
      - TARGET: ["target/*.jar", "$IMAGE:$TAG"]
  extends: .rhtas-verify
  variables:
    TARGET_PATH: $TARGET

deploy:
  stage: deploy
  script:
    - deploy_to_production
  dependencies:
    - verify-signatures
  only:
    - main
```

## Advanced Configuration

### Custom Verification Policy

```yaml
verify-strict:
  extends: .rhtas-verify
  variables:
    VERIFICATION_POLICY: |
      {
        "type": "strict",
        "required_signatures": 2,
        "trusted_identities": [
          "signer1@example.com",
          "signer2@example.com"
        ],
        "required_attestations": [
          "security-scan",
          "quality-gate"
        ]
      }
```

### Environment-Specific Configuration

```yaml
.sign-base:
  extends: .rhtas-sign-artifact
  variables:
    RHTAS_URL: "https://rhtas.example.com"

sign-development:
  extends: .sign-base
  variables:
    RHTAS_OIDC_ISSUER: "https://gitlab-dev.example.com"
  only:
    - develop

sign-production:
  extends: .sign-base
  variables:
    RHTAS_OIDC_ISSUER: "https://gitlab-prod.example.com"
  only:
    - main
```