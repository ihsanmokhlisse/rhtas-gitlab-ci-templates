# Examples

Practical examples of RHTAS GitLab CI template implementations.

## Maven Artifact Signing

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

verify:
  stage: verify
  extends: .rhtas-verify
  variables:
    TARGET_PATH: "target/*.jar"
```

## Container Image Signing

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
```

## Multi-Stage Pipeline

```yaml
include:
  - remote: 'https://raw.githubusercontent.com/ihsanmokhlisse/rhtas-gitlab-ci-templates/main/templates/artifact-signing.yml'
  - remote: 'https://raw.githubusercontent.com/ihsanmokhlisse/rhtas-gitlab-ci-templates/main/templates/container-signing.yml'

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

test:
  stage: test
  script:
    - mvn test

sign-artifact:
  stage: sign
  extends: .rhtas-sign-artifact
  variables:
    ARTIFACT_PATH: "target/*.jar"

verify-signatures:
  stage: verify
  extends: .rhtas-verify
  variables:
    TARGET_PATH: "target/*.jar"

deploy:
  stage: deploy
  script:
    - deploy_to_production
  only:
    - main
```