---
name: ci-cd-pipeline-builder
description: Creates CI/CD pipelines for GitHub Actions, GitLab CI, or CircleCI with quality gates, automated testing, security scanning, and deployment strategies. Use when setting up new projects OR improving deployment automation OR implementing quality gates.
allowed-tools: Read, Write, Edit, Bash
---

# CI/CD Pipeline Builder

You build comprehensive CI/CD pipelines with automated testing, quality gates, security scanning, and safe deployment strategies.

## When to use
- Setting up a new project
- Migrating from manual deployments
- Implementing quality gates
- Adding automated security scanning
- Setting up multi-environment deployments
- Improving deployment safety

## Pipeline Stages

### 1. Build
Compile/build code, install dependencies.

### 2. Test
Run unit, integration, and e2e tests.

### 3. Quality Gates
Linting, type checking, coverage, security scanning.

### 4. Build Artifacts
Create Docker images, build packages.

### 5. Deploy
Deploy to staging/production with health checks.

### 6. Post-Deploy
Smoke tests, notifications.

## GitHub Actions (Recommended)

### Basic Pipeline

```yaml
# .github/workflows/ci.yml
name: CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

env:
  NODE_VERSION: '18'
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  test:
    runs-on: ubuntu-latest

    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_PASSWORD: postgres
          POSTGRES_DB: test_db
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 5432:5432

      redis:
        image: redis:7-alpine
        ports:
          - 6379:6379

    steps:
      - uses: actions/checkout@v3

      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Run linter
        run: npm run lint

      - name: Type check
        run: npm run typecheck

      - name: Run unit tests
        run: npm run test:unit
        env:
          DATABASE_URL: postgresql://postgres:postgres@localhost:5432/test_db
          REDIS_URL: redis://localhost:6379

      - name: Run integration tests
        run: npm run test:integration
        env:
          DATABASE_URL: postgresql://postgres:postgres@localhost:5432/test_db
          REDIS_URL: redis://localhost:6379

      - name: Generate coverage report
        run: npm run test:coverage

      - name: Upload coverage to Codecov
        uses: codecov/codecov-action@v3
        with:
          files: ./coverage/coverage-final.json
          fail_ci_if_error: true

      - name: Check coverage threshold
        run: |
          COVERAGE=$(node -e "console.log(require('./coverage/coverage-summary.json').total.lines.pct)")
          if (( $(echo "$COVERAGE < 80" | bc -l) )); then
            echo "Coverage $COVERAGE% is below 80%"
            exit 1
          fi

  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Run npm audit
        run: npm audit --audit-level=high

      - name: Run Snyk security scan
        uses: snyk/actions/node@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
        with:
          args: --severity-threshold=high

      - name: Run Trivy vulnerability scanner
        uses: aquasecurity/trivy-action@master
        with:
          scan-type: 'fs'
          scan-ref: '.'
          format: 'sarif'
          output: 'trivy-results.sarif'

      - name: Upload Trivy results to GitHub Security
        uses: github/codeql-action/upload-sarif@v2
        with:
          sarif_file: 'trivy-results.sarif'

  build:
    needs: [test, security]
    runs-on: ubuntu-latest
    if: github.event_name == 'push'

    steps:
      - uses: actions/checkout@v3

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v2

      - name: Log in to Container Registry
        uses: docker/login-action@v2
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v4
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
          tags: |
            type=ref,event=branch
            type=ref,event=pr
            type=semver,pattern={{version}}
            type=semver,pattern={{major}}.{{minor}}
            type=sha

      - name: Build and push Docker image
        uses: docker/build-push-action@v4
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max

  deploy-staging:
    needs: build
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    environment:
      name: staging
      url: https://staging.example.com

    steps:
      - uses: actions/checkout@v3

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v2
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1

      - name: Deploy to ECS
        run: |
          aws ecs update-service \
            --cluster staging \
            --service api \
            --force-new-deployment

      - name: Wait for deployment
        run: |
          aws ecs wait services-stable \
            --cluster staging \
            --services api

      - name: Run smoke tests
        run: |
          curl -f https://staging.example.com/health || exit 1

      - name: Notify Slack
        uses: slackapi/slack-github-action@v1
        with:
          payload: |
            {
              "text": "Deployed to staging: ${{ github.sha }}"
            }
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}

  deploy-production:
    needs: deploy-staging
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    environment:
      name: production
      url: https://example.com

    steps:
      - uses: actions/checkout@v3

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v2
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1

      - name: Deploy to production (blue-green)
        run: |
          # Update task definition with new image
          TASK_DEF=$(aws ecs describe-task-definition --task-definition api --query taskDefinition)
          NEW_TASK_DEF=$(echo $TASK_DEF | jq --arg IMAGE "${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }}" '.containerDefinitions[0].image = $IMAGE')

          # Register new task definition
          aws ecs register-task-definition --cli-input-json "$NEW_TASK_DEF"

          # Update service
          aws ecs update-service \
            --cluster production \
            --service api \
            --task-definition api

      - name: Monitor deployment
        run: |
          for i in {1..30}; do
            ERROR_RATE=$(curl -s https://api.example.com/metrics | grep error_rate | awk '{print $2}')
            if (( $(echo "$ERROR_RATE > 0.05" | bc -l) )); then
              echo "Error rate too high: $ERROR_RATE"
              exit 1
            fi
            sleep 10
          done

      - name: Notify team
        uses: slackapi/slack-github-action@v1
        with:
          payload: |
            {
              "text": "✅ Deployed to production: ${{ github.sha }}\nURL: https://example.com"
            }
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
```

### Matrix Testing (Multiple Versions)

```yaml
jobs:
  test:
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, macos-latest, windows-latest]
        node-version: [16, 18, 20]
        exclude:
          - os: macos-latest
            node-version: 16

    steps:
      - uses: actions/checkout@v3

      - name: Setup Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v3
        with:
          node-version: ${{ matrix.node-version }}

      - run: npm test
```

### Monorepo with Changed Files Detection

```yaml
jobs:
  changes:
    runs-on: ubuntu-latest
    outputs:
      api: ${{ steps.filter.outputs.api }}
      web: ${{ steps.filter.outputs.web }}
    steps:
      - uses: actions/checkout@v3
      - uses: dorny/paths-filter@v2
        id: filter
        with:
          filters: |
            api:
              - 'packages/api/**'
            web:
              - 'packages/web/**'

  test-api:
    needs: changes
    if: needs.changes.outputs.api == 'true'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - run: npm test --workspace=api

  test-web:
    needs: changes
    if: needs.changes.outputs.web == 'true'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - run: npm test --workspace=web
```

## GitLab CI

```yaml
# .gitlab-ci.yml
stages:
  - test
  - build
  - deploy

variables:
  NODE_VERSION: "18"
  DOCKER_DRIVER: overlay2

# Template for Node.js setup
.node-template: &node-template
  image: node:${NODE_VERSION}
  cache:
    paths:
      - node_modules/
  before_script:
    - npm ci

test:unit:
  <<: *node-template
  stage: test
  services:
    - postgres:15
    - redis:7-alpine
  variables:
    POSTGRES_DB: test_db
    POSTGRES_USER: postgres
    POSTGRES_PASSWORD: postgres
    DATABASE_URL: postgresql://postgres:postgres@postgres:5432/test_db
    REDIS_URL: redis://redis:6379
  script:
    - npm run test:unit
    - npm run test:integration
  coverage: '/All files\s+\|\s+([\d\.]+)/'
  artifacts:
    reports:
      coverage_report:
        coverage_format: cobertura
        path: coverage/cobertura-coverage.xml

test:lint:
  <<: *node-template
  stage: test
  script:
    - npm run lint
    - npm run typecheck

security:scan:
  stage: test
  image: aquasec/trivy:latest
  script:
    - trivy fs --exit-code 1 --severity HIGH,CRITICAL .

build:docker:
  stage: build
  image: docker:latest
  services:
    - docker:dind
  before_script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
  script:
    - docker build -t $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA .
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
    - docker tag $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA $CI_REGISTRY_IMAGE:latest
    - docker push $CI_REGISTRY_IMAGE:latest
  only:
    - main

deploy:staging:
  stage: deploy
  image: alpine:latest
  before_script:
    - apk add --no-cache curl
  script:
    - curl -X POST $DEPLOY_WEBHOOK_STAGING
    - sleep 30
    - curl -f https://staging.example.com/health || exit 1
  environment:
    name: staging
    url: https://staging.example.com
  only:
    - main

deploy:production:
  stage: deploy
  image: alpine:latest
  before_script:
    - apk add --no-cache curl
  script:
    - curl -X POST $DEPLOY_WEBHOOK_PRODUCTION
    - sleep 30
    - curl -f https://example.com/health || exit 1
  environment:
    name: production
    url: https://example.com
  when: manual
  only:
    - main
```

## CircleCI

```yaml
# .circleci/config.yml
version: 2.1

orbs:
  node: circleci/node@5.0
  docker: circleci/docker@2.0
  aws-ecs: circleci/aws-ecs@3.0

executors:
  node-executor:
    docker:
      - image: cimg/node:18.17
        environment:
          DATABASE_URL: postgresql://postgres:postgres@localhost:5432/test_db
          REDIS_URL: redis://localhost:6379
      - image: postgres:15
        environment:
          POSTGRES_PASSWORD: postgres
          POSTGRES_DB: test_db
      - image: redis:7-alpine

jobs:
  test:
    executor: node-executor
    steps:
      - checkout
      - node/install-packages
      - run:
          name: Run linter
          command: npm run lint
      - run:
          name: Run type check
          command: npm run typecheck
      - run:
          name: Run tests
          command: npm test
      - run:
          name: Check coverage
          command: |
            COVERAGE=$(node -e "console.log(require('./coverage/coverage-summary.json').total.lines.pct)")
            if (( $(echo "$COVERAGE < 80" | bc -l) )); then
              echo "Coverage $COVERAGE% is below 80%"
              exit 1
            fi
      - store_test_results:
          path: ./test-results
      - store_artifacts:
          path: ./coverage

  security:
    docker:
      - image: cimg/node:18.17
    steps:
      - checkout
      - run:
          name: Run npm audit
          command: npm audit --audit-level=high
      - run:
          name: Run Snyk
          command: |
            npm install -g snyk
            snyk test --severity-threshold=high

  build:
    executor: docker/docker
    steps:
      - checkout
      - setup_remote_docker
      - docker/check
      - docker/build:
          image: myorg/myapp
          tag: ${CIRCLE_SHA1}
      - docker/push:
          image: myorg/myapp
          tag: ${CIRCLE_SHA1}

  deploy-staging:
    executor: aws-ecs/default
    steps:
      - aws-ecs/update-service:
          cluster: staging
          service-name: api
          force-new-deployment: true
      - run:
          name: Smoke test
          command: curl -f https://staging.example.com/health || exit 1

  deploy-production:
    executor: aws-ecs/default
    steps:
      - aws-ecs/update-service:
          cluster: production
          service-name: api
          force-new-deployment: true
      - run:
          name: Health check
          command: curl -f https://example.com/health || exit 1

workflows:
  build-test-deploy:
    jobs:
      - test
      - security
      - build:
          requires:
            - test
            - security
          filters:
            branches:
              only: main
      - deploy-staging:
          requires:
            - build
      - hold-production:
          type: approval
          requires:
            - deploy-staging
      - deploy-production:
          requires:
            - hold-production
```

## Quality Gates

### Code Coverage
```yaml
- name: Check coverage threshold
  run: |
    COVERAGE=$(jq '.total.lines.pct' coverage/coverage-summary.json)
    if (( $(echo "$COVERAGE < 80" | bc -l) )); then
      echo "::error::Coverage ${COVERAGE}% is below 80%"
      exit 1
    fi
```

### Dependency Vulnerabilities
```yaml
- name: Audit dependencies
  run: npm audit --audit-level=moderate
```

### Code Quality (SonarQube)
```yaml
- name: SonarQube Scan
  uses: sonarsource/sonarqube-scan-action@master
  env:
    SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
    SONAR_HOST_URL: ${{ secrets.SONAR_HOST_URL }}
```

### Bundle Size
```yaml
- name: Check bundle size
  run: |
    SIZE=$(du -sk dist | cut -f1)
    if [ $SIZE -gt 1000 ]; then
      echo "::error::Bundle size ${SIZE}KB exceeds 1MB"
      exit 1
    fi
```

## Deployment Strategies

### Blue-Green Deployment
```yaml
- name: Blue-Green Deploy
  run: |
    # Deploy to green environment
    aws ecs update-service --cluster prod --service api-green --force-new-deployment

    # Wait for healthy
    aws ecs wait services-stable --cluster prod --services api-green

    # Switch traffic
    aws elbv2 modify-rule --rule-arn $RULE_ARN --actions TargetGroupArn=api-green

    # Monitor for 5 minutes
    sleep 300

    # If successful, decommission blue
    aws ecs update-service --cluster prod --service api-blue --desired-count 0
```

### Canary Deployment
```yaml
- name: Canary Deploy
  run: |
    # Deploy canary (10% traffic)
    aws ecs update-service --cluster prod --service api-canary --desired-count 1

    # Monitor metrics for 10 minutes
    ./scripts/monitor-canary.sh

    # If successful, roll out to 50%
    aws ecs update-service --cluster prod --service api-canary --desired-count 5

    # Monitor again
    ./scripts/monitor-canary.sh

    # If successful, full rollout
    aws ecs update-service --cluster prod --service api --force-new-deployment
```

## Rollback

```yaml
- name: Automatic rollback on failure
  if: failure()
  run: |
    # Rollback to previous task definition
    aws ecs update-service \
      --cluster production \
      --service api \
      --task-definition api:PREVIOUS

    # Notify team
    curl -X POST $SLACK_WEBHOOK -d '{"text":"🚨 Deployment failed, rolling back"}'
```

## Notifications

```yaml
- name: Notify on success
  if: success()
  uses: slackapi/slack-github-action@v1
  with:
    payload: |
      {
        "text": "✅ Deployment successful",
        "blocks": [
          {
            "type": "section",
            "text": {
              "type": "mrkdwn",
              "text": "*Deployment Successful*\n<${{ github.server_url }}/${{ github.repository }}/commit/${{ github.sha }}|${{ github.sha }}>\nby ${{ github.actor }}"
            }
          }
        ]
      }
  env:
    SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}

- name: Notify on failure
  if: failure()
  uses: slackapi/slack-github-action@v1
  with:
    payload: |
      {
        "text": "❌ Deployment failed - <${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}|View logs>"
      }
  env:
    SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
```

## Best Practices

✅ **DO:**
- Run tests in parallel when possible
- Cache dependencies
- Use matrix builds for multi-version support
- Implement quality gates (coverage, security)
- Deploy to staging before production
- Use manual approval for production
- Monitor deployments
- Set up automatic rollback
- Notify team of deployments

❌ **DON'T:**
- Deploy directly to production
- Skip tests to "move faster"
- Hardcode secrets in pipelines
- Deploy without health checks
- Ignore failing tests
- Deploy outside business hours (without on-call)

## Instructions

1. **Choose platform**: GitHub Actions, GitLab CI, CircleCI
2. **Define stages**: test → build → deploy
3. **Add quality gates**: coverage, linting, security
4. **Configure environments**: staging, production
5. **Set up secrets**: AWS keys, tokens, etc.
6. **Test pipeline**: Run on feature branch
7. **Add notifications**: Slack, email
8. **Document process**: README section on CI/CD

## Constraints

- Must run tests before deployment
- Must not expose secrets
- Must deploy to staging before production
- Must have manual approval for production
- Must include health checks
- Must notify team of deployments
- Must support rollback
