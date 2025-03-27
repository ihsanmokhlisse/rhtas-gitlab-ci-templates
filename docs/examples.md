# RHTAS Integration Examples

## Maven Artifact Signing

```yaml
include:
  - remote: 'https://raw.githubusercontent.com/ihsanmokhlisse/rhtas-gitlab-ci-templates/main/templates/artifact-signing.yml'

variables:
  RHTAS_URL: "https://rhtas.example.com"
  RHTAS_OIDC_ISSUER: "https://gitlab.example.com"

build:
  image: maven:3.8-openjdk-11
  script:
    - mvn package
  artifacts:
    paths:
      - target/*.jar

sign:
  extends: .rhtas-sign-artifact
  needs: [build]
  variables:
    ARTIFACT_PATH: "target/*.jar"
```

## Container Image Signing

```yaml
include:
  - remote: 'https://raw.githubusercontent.com/ihsanmokhlisse/rhtas-gitlab-ci-templates/main/templates/container-signing.yml'

variables:
  RHTAS_URL: "https://rhtas.example.com"
  RHTAS_OIDC_ISSUER: "https://gitlab.example.com"

build:
  image: docker:20.10
  services:
    - docker:20.10-dind
  script:
    - docker build -t $CI_REGISTRY_IMAGE:$CI_COMMIT_TAG .
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_TAG

sign:
  extends: .rhtas-sign-container
  needs: [build]
  variables:
    CONTAINER_IMAGE: "${CI_REGISTRY_IMAGE}:${CI_COMMIT_TAG}"
```

## Signature Verification

```yaml
include:
  - remote: 'https://raw.githubusercontent.com/ihsanmokhlisse/rhtas-gitlab-ci-templates/main/templates/verification.yml'

verify:
  extends: .rhtas-verify
  variables:
    TARGET_PATH: "${CI_REGISTRY_IMAGE}:${CI_COMMIT_TAG}"
    VERIFICATION_POLICY: "strict"
```