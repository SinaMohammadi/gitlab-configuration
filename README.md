# GitLab CI/CD Configuration Documentation

## Overview

This the CI/CD pipeline configuration for this project, implemented using GitLab CI/CD. The pipeline automates building, testing, security scanning, and deployment processes.

## Pipeline Structure

### Stages
The pipeline consists of four main stages that run sequentially:
1. `build`: Compiles and prepares the application
2. `test`: Runs unit and integration tests
3. `security`: Performs security scans
4. `deploy`: Handles deployment to staging and production environments

### Base Configuration

```yaml
image: node:16

cache:
  key: ${CI_COMMIT_REF_SLUG}
  paths:
    - node_modules/
```

- Uses Node.js 16 as the base image
- Implements caching for node_modules to speed up pipeline execution

### Security Templates

```yaml
include:
  - template: Security/SAST.gitlab-ci.yml
  - template: Security/Dependency-Scanning.gitlab-ci.yml
```

Incorporates GitLab's built-in security scanning templates:
- Static Application Security Testing (SAST)
- Dependency Scanning

## Job Configurations

### Build Stage

```yaml
build:
  stage: build
  script:
    - npm install
    - npm run build
  artifacts:
    paths:
      - dist/
    expire_in: 1 week
```

- Installs dependencies
- Builds the application
- Stores build artifacts for 1 week

### Test Stage

```yaml
unit_tests:
  stage: test
  script:
    - npm run test:unit
  coverage: '/Coverage: \d+\.\d+%/'

integration_tests:
  stage: test
  script:
    - npm run test:integration
  allow_failure: true
```

- Runs unit tests with coverage reporting
- Executes integration tests (allowed to fail)

### Security Stage

```yaml
sast:
  stage: security
  extends: .sast

dependency_scanning:
  stage: security
```

- Performs static code analysis
- Scans dependencies for known vulnerabilities

### Deploy Stage

```yaml
deploy_staging:
  stage: deploy
  script:
    - npm run deploy:staging
  environment:
    name: staging
  only:
    - develop

deploy_production:
  stage: deploy
  script:
    - npm run deploy:production
  environment:
    name: production
  only:
    - main
  when: manual
```

- Automatically deploys to staging from develop branch
- Manually triggers production deployment from main branch

## Pipeline Rules

```yaml
workflow:
  rules:
    - if: $CI_COMMIT_BRANCH != "main" && $CI_PIPELINE_SOURCE != "merge_request_event"
      when: never
    - when: always
```

- Controls when pipelines are triggered
- Prevents unnecessary pipeline runs

## Variables

```yaml
variables:
  NODE_ENV: "development"
```

Default environment variables available to all jobs.

## Customization Guide

### Adding New Jobs

To add a new job:

```yaml
job_name:
  stage: existing_stage
  script:
    - your commands here
  rules:
    - if: condition
      when: on_success
```

### Modifying Stages

To add new stages, update the stages list:

```yaml
stages:
  - build
  - test
  - security
  - your_new_stage
  - deploy
```


## Support and Resources

- [GitLab CI/CD Documentation](https://docs.gitlab.com/ee/ci/)
- [GitLab CI/CD Variables Reference](https://docs.gitlab.com/ee/ci/variables/predefined_variables.html)
- [Security Template Documentation](https://docs.gitlab.com/ee/user/application_security/)
