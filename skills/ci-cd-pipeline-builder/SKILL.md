---
name: ci-cd-pipeline-builder
description: Creates complete CI/CD pipelines for GitHub Actions, GitLab CI, CircleCI, or Jenkins with automated testing, code quality gates (linting, type checking), security scanning (Snyk, Trivy), Docker builds, and multi-environment deployments (dev/staging/prod). Generates workflow YAML files with caching, parallelization, and deployment strategies. Use when setting up new projects, automating deployments, implementing quality gates, adding security scans, containerizing applications, or setting up monorepo pipelines.
allowed-tools: Read, Write, Edit, Bash
---

# CI/CD Pipeline Builder

You build comprehensive CI/CD pipelines with automated testing, quality gates, security scanning, and safe deployment strategies.

## When to use
- Setting up a new project's CI/CD
- Migrating from manual deployments
- Implementing automated quality gates
- Adding security scanning to pipelines
- Setting up multi-environment deployments (staging, production)
- Improving deployment safety and reliability
- Adding automated rollback capabilities

## Pipeline Stages

A good CI/CD pipeline has these stages:

### 1. Build
- Install dependencies
- Compile/transpile code
- Generate assets

### 2. Test
- Run unit tests
- Run integration tests
- Run end-to-end tests
- Generate coverage reports

### 3. Quality Gates
- Linting (code style)
- Type checking
- Code coverage threshold
- Security scanning
- Dependency vulnerability checks

### 4. Build Artifacts
- Create Docker images
- Build deployment packages
- Tag with version/commit SHA

### 5. Deploy
- Deploy to staging (automatic)
- Deploy to production (manual approval or automated)
- Health checks after deployment
- Smoke tests

### 6. Post-Deploy
- Run smoke tests
- Send notifications
- Monitor metrics

## Basic GitHub Actions Pipeline

```yaml
# .github/workflows/ci.yml
name: CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Run linter
        run: npm run lint

      - name: Type check
        run: npm run typecheck

      - name: Run tests
        run: npm test

      - name: Check coverage
        run: npm run test:coverage

  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Run security audit
        run: npm audit --audit-level=high

      - name: Scan vulnerabilities
        uses: snyk/actions/node@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}

  build:
    needs: [test, security]
    runs-on: ubuntu-latest
    if: github.event_name == 'push'

    steps:
      - uses: actions/checkout@v3

      - name: Build Docker image
        run: docker build -t myapp:${{ github.sha }} .

      - name: Push to registry
        run: docker push myapp:${{ github.sha }}

  deploy:
    needs: build
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    environment: production

    steps:
      - name: Deploy to production
        run: ./deploy.sh ${{ github.sha }}
```

**Key concepts:**
- `needs:` creates dependencies between jobs
- `if:` conditionally runs jobs
- `environment:` adds manual approval gates
- Jobs run in parallel unless `needs` is specified

## Platform-Specific Guides

For detailed platform implementations, see:

### CI/CD Platforms
- **GitHub Actions:** `examples/github-actions.md`
  - Complete workflow examples
  - Matrix builds
  - Reusable workflows
  - GitHub Container Registry
  - Deployment environments

- **GitLab CI:** `examples/gitlab-ci.md`
  - .gitlab-ci.yml structure
  - Stages and jobs
  - GitLab Runner setup
  - Container Registry
  - Auto DevOps

- **CircleCI:** `examples/circleci.md`
  - config.yml structure
  - Workflows and jobs
  - Docker executor
  - Caching strategies

### Advanced Topics
- **Quality Gates:** `examples/quality-gates.md`
  - Code coverage enforcement
  - Linting and formatting
  - Security scanning tools
  - Performance budgets

- **Deployment Strategies:** `examples/deployment-strategies.md`
  - Blue-green deployments
  - Canary deployments
  - Rolling updates
  - Feature flags

- **Rollback:** `examples/rollback.md`
  - Automatic rollback triggers
  - Manual rollback procedures
  - Database migration rollback

### Templates
- **GitHub Actions:** `templates/github-workflow.yml`
- **GitLab CI:** `templates/gitlab-ci.yml`
- **CircleCI:** `templates/circleci-config.yml`

## Instructions

When building a CI/CD pipeline:

1. **Choose platform:**
   - GitHub Actions (if using GitHub)
   - GitLab CI (if using GitLab)
   - CircleCI (for flexible cloud CI)

2. **Define stages:**
   - Start with: test → build → deploy
   - Add quality gates
   - Add security scanning
   - Add deployment approvals

3. **Implement quality gates:**
   - Linting must pass
   - Type checking must pass
   - Tests must pass
   - Coverage must meet threshold (e.g., 80%)
   - Security scans must pass

4. **Add security scanning:**
   - Dependency vulnerability scanning (npm audit, Snyk)
   - Container scanning (Trivy, Grype)
   - SAST scanning (CodeQL, SonarQube)
   - Secret scanning

5. **Configure deployments:**
   - Staging: automatic on main branch
   - Production: manual approval or tag-based
   - Health checks after deploy
   - Automatic rollback on failure

6. **Add notifications:**
   - Slack/Discord on failures
   - Email on deployment
   - GitHub commit status

7. **Optimize for speed:**
   - Cache dependencies
   - Run jobs in parallel
   - Use matrix builds for multi-platform
   - Skip redundant steps

## Common Patterns

### Multi-Environment Deployment

```yaml
deploy-staging:
  if: github.ref == 'refs/heads/develop'
  steps:
    - name: Deploy to staging
      run: ./deploy.sh staging

deploy-production:
  if: github.ref == 'refs/heads/main'
  environment: production  # Requires manual approval
  steps:
    - name: Deploy to production
      run: ./deploy.sh production
```

### Matrix Builds

```yaml
test:
  strategy:
    matrix:
      node-version: [16, 18, 20]
      os: [ubuntu-latest, macos-latest]
  runs-on: ${{ matrix.os }}
  steps:
    - uses: actions/setup-node@v3
      with:
        node-version: ${{ matrix.node-version }}
```

### Caching Dependencies

```yaml
- name: Cache dependencies
  uses: actions/cache@v3
  with:
    path: ~/.npm
    key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
    restore-keys: |
      ${{ runner.os }}-node-
```

### Coverage Threshold

```yaml
- name: Check coverage threshold
  run: |
    COVERAGE=$(node -e "console.log(require('./coverage/coverage-summary.json').total.lines.pct)")
    if (( $(echo "$COVERAGE < 80" | bc -l) )); then
      echo "Coverage $COVERAGE% is below 80%"
      exit 1
    fi
```

## Best Practices

✅ **DO:**
- Run tests before deploying
- Use quality gates (linting, coverage, security)
- Cache dependencies for speed
- Use semantic versioning for releases
- Add manual approval for production
- Implement health checks after deploy
- Set up automatic rollback
- Use secrets management (never commit secrets)
- Run security scans on every build
- Test in staging before production
- Use infrastructure as code
- Version your pipeline configs

❌ **DON'T:**
- Deploy without running tests
- Skip security scanning
- Hardcode secrets in configs
- Deploy directly to production
- Ignore test failures
- Skip code quality checks
- Deploy without health checks
- Use long-running self-hosted runners for public repos
- Allow unverified dependencies
- Skip rollback planning

## Security Considerations

**Secrets Management:**
```yaml
- name: Deploy
  env:
    API_KEY: ${{ secrets.API_KEY }}
    DB_PASSWORD: ${{ secrets.DB_PASSWORD }}
  run: ./deploy.sh
```

**Dependency Scanning:**
```yaml
- name: Audit dependencies
  run: npm audit --audit-level=high

- name: Snyk security scan
  uses: snyk/actions/node@master
  env:
    SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
```

**Container Scanning:**
```yaml
- name: Scan Docker image
  uses: aquasecurity/trivy-action@master
  with:
    image-ref: 'myapp:latest'
    severity: 'CRITICAL,HIGH'
```

## Deployment Strategies

**Blue-Green:**
```yaml
- name: Deploy to green environment
  run: ./deploy.sh green

- name: Run smoke tests on green
  run: ./smoke-tests.sh green

- name: Switch traffic to green
  run: ./switch-traffic.sh green

- name: Keep blue as rollback
  run: echo "Blue environment ready for rollback"
```

**Canary:**
```yaml
- name: Deploy canary (10% traffic)
  run: ./deploy-canary.sh 10

- name: Monitor metrics
  run: ./monitor.sh --duration=10m

- name: Roll out to 100%
  if: success()
  run: ./deploy-canary.sh 100
```

## Rollback

**Automatic rollback on health check failure:**
```yaml
- name: Deploy
  run: ./deploy.sh ${{ github.sha }}

- name: Health check
  run: |
    if ! ./health-check.sh; then
      echo "Health check failed, rolling back"
      ./rollback.sh
      exit 1
    fi
```

## Notifications

**Slack notification:**
```yaml
- name: Notify Slack on failure
  if: failure()
  uses: slackapi/slack-github-action@v1
  with:
    payload: |
      {
        "text": "Build failed: ${{ github.sha }}"
      }
  env:
    SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK }}
```

## Constraints

- Must run all tests before deploying
- Must implement quality gates
- Must scan for security vulnerabilities
- Must use secrets management (no hardcoded secrets)
- Production deployments must require approval or tag
- Must implement health checks post-deployment
- Must have rollback capability
- Must cache dependencies for performance
- Must use proper branching strategy
- Must version artifacts (Docker images, packages)
