---
name: documentation-generator
description: Generates comprehensive documentation including README files, architecture diagrams, ADRs, API docs, onboarding guides, and deployment runbooks from code and project structure.
allowed-tools: Read, Write, Grep, Glob, Bash
---

# Documentation Generator

You generate comprehensive, maintainable documentation that helps teams understand, use, and contribute to projects.

## When to use
- Starting a new project (initial README)
- Onboarding new team members
- Creating deployment runbooks
- Documenting architectural decisions (ADRs)
- Generating API documentation
- Creating architecture diagrams
- Writing contributing guidelines

## Documentation Types

### 1. README.md (Project Overview)
Primary entry point for any project.

### 2. Architecture Decision Records (ADRs)
Document why decisions were made.

### 3. API Documentation
Endpoint documentation, examples.

### 4. Onboarding Guide
Get new developers productive fast.

### 5. Deployment Runbook
Step-by-step deployment instructions.

### 6. Contributing Guidelines
How to contribute to the project.

### 7. Architecture Diagrams
Visual system overview.

## README.md Template

```markdown
# Project Name

Brief description of what this project does and why it exists.

## Features

- ✨ Feature 1
- 🚀 Feature 2
- 🔒 Feature 3

## Tech Stack

- **Backend**: FastAPI, PostgreSQL, Redis
- **Frontend**: React, TypeScript, Tailwind CSS
- **Infrastructure**: Docker, AWS ECS, Terraform
- **Monitoring**: Prometheus, Grafana, Sentry

## Prerequisites

- Node.js 18+ or Python 3.11+
- Docker and Docker Compose
- PostgreSQL 15+
- (Optional) AWS CLI for deployment

## Quick Start

### Option 1: Docker (Recommended)

\`\`\`bash
# Clone the repository
git clone https://github.com/org/project.git
cd project

# Copy environment file
cp .env.example .env

# Start services
docker-compose up -d

# Run migrations
docker-compose exec api python manage.py migrate

# Access the app
open http://localhost:3000
\`\`\`

### Option 2: Local Development

\`\`\`bash
# Install dependencies
npm install  # or: pip install -r requirements.txt

# Set up database
createdb myapp_dev
npm run migrate  # or: alembic upgrade head

# Start development server
npm run dev  # or: uvicorn main:app --reload

# Access the app
open http://localhost:3000
\`\`\`

## Project Structure

\`\`\`
project/
├── src/
│   ├── api/           # API endpoints
│   ├── models/        # Data models
│   ├── services/      # Business logic
│   └── utils/         # Utilities
├── tests/
│   ├── unit/          # Unit tests
│   └── integration/   # Integration tests
├── docs/              # Documentation
├── docker-compose.yml
└── README.md
\`\`\`

## Configuration

### Environment Variables

Required:
- `DATABASE_URL` - PostgreSQL connection string
- `REDIS_URL` - Redis connection string
- `JWT_SECRET` - Secret for JWT tokens (min 32 chars)

Optional:
- `LOG_LEVEL` - Logging level (default: info)
- `PORT` - Server port (default: 8000)

See `.env.example` for full list.

## Development

### Running Tests

\`\`\`bash
# All tests
npm test

# Unit tests only
npm run test:unit

# Integration tests
npm run test:integration

# With coverage
npm run test:coverage
\`\`\`

### Code Quality

\`\`\`bash
# Linting
npm run lint

# Type checking
npm run typecheck

# Formatting
npm run format
\`\`\`

### Database Migrations

\`\`\`bash
# Create migration
npm run migrate:create -- add_users_table

# Run migrations
npm run migrate

# Rollback
npm run migrate:rollback
\`\`\`

## Deployment

See [DEPLOYMENT.md](docs/DEPLOYMENT.md) for detailed deployment instructions.

### Quick Deploy

\`\`\`bash
# Build
npm run build

# Deploy to staging
npm run deploy:staging

# Deploy to production
npm run deploy:production
\`\`\`

## API Documentation

API docs available at:
- Development: http://localhost:8000/docs
- Staging: https://api-staging.example.com/docs
- Production: https://api.example.com/docs

## Architecture

See [ARCHITECTURE.md](docs/ARCHITECTURE.md) for system architecture overview.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for contribution guidelines.

## License

MIT License - see [LICENSE](LICENSE) for details.

## Support

- 📧 Email: support@example.com
- 💬 Slack: #project-support
- 📖 Docs: https://docs.example.com
- 🐛 Issues: https://github.com/org/project/issues

## Acknowledgments

- Thanks to [contributor](https://github.com/contributor)
- Built with [awesome-lib](https://github.com/awesome-lib)
\`\`\`

## Architecture Decision Record (ADR)

```markdown
# ADR-0001: Use PostgreSQL for Primary Database

**Date**: 2025-01-24
**Status**: Accepted
**Deciders**: Alice (Tech Lead), Bob (Backend Dev)

## Context

We need to choose a database for our application that handles user data, transactions, and relationships.

### Requirements
- ACID compliance for financial transactions
- Support for complex queries and joins
- Mature ecosystem and tooling
- Good performance for read-heavy workloads
- Support for JSON data (flexible schema)

## Decision

We will use **PostgreSQL 15** as our primary database.

## Rationale

### Pros
- **ACID compliance**: Critical for financial data integrity
- **Mature and stable**: 30+ years of development
- **Rich feature set**: JSON support, full-text search, geospatial
- **Performance**: Excellent for complex queries and analytical workloads
- **Ecosystem**: ORMs (SQLAlchemy, TypeORM), migration tools, monitoring
- **Open source**: No vendor lock-in, large community

### Alternatives Considered

**MySQL**
- Pros: Simpler, slightly better for write-heavy workloads
- Cons: Less feature-rich, weaker JSON support
- Why not: We need advanced features (CTEs, window functions)

**MongoDB**
- Pros: Flexible schema, horizontal scaling
- Cons: No transactions (at the time), eventual consistency
- Why not: Need ACID guarantees for financial data

**DynamoDB**
- Pros: Managed, auto-scaling, serverless
- Cons: Vendor lock-in, limited query flexibility, expensive for reads
- Why not: Complex queries required, want multi-cloud option

## Consequences

### Positive
- Reliable, well-tested database for critical data
- Rich querying capabilities (CTEs, window functions, JSON)
- Excellent tooling and community support
- Can use JSONB for semi-structured data

### Negative
- Vertical scaling (need bigger servers, not just add more)
- More complex to operate than managed services
- Requires expertise for performance tuning

### Neutral
- Need to set up replication for HA
- Will use connection pooling (PgBouncer)
- Regular backups and point-in-time recovery needed

## Implementation Notes

- Use Docker for local development
- RDS for staging/production (managed)
- Connection pooling with PgBouncer
- Read replicas for analytics queries
- Backup retention: 30 days
- Monitoring: CloudWatch + custom Prometheus metrics

## Related Decisions

- ADR-0002: Use SQLAlchemy as ORM
- ADR-0003: Database migration strategy (Alembic)

## References

- [PostgreSQL Documentation](https://www.postgresql.org/docs/)
- [Comparison: PostgreSQL vs MySQL](https://example.com/comparison)
- Internal discussion: [Link to Slack thread]
```

## Deployment Runbook

```markdown
# Deployment Runbook

## Overview

This runbook describes how to deploy the application to staging and production environments.

## Environments

| Environment | URL | Purpose |
|-------------|-----|---------|
| Development | http://localhost:3000 | Local development |
| Staging | https://staging.example.com | Pre-production testing |
| Production | https://example.com | Live users |

## Pre-Deployment Checklist

- [ ] All tests passing (`npm test`)
- [ ] Code reviewed and approved
- [ ] Database migrations tested on staging
- [ ] Environment variables configured
- [ ] Monitoring alerts configured
- [ ] Rollback plan documented

## Deployment Process

### Staging Deployment

**Trigger**: Merge to `main` branch (automatic)

**Steps**:
1. CI/CD runs tests
2. Build Docker image
3. Push to container registry
4. Deploy to ECS staging cluster
5. Run database migrations
6. Health check verification
7. Smoke tests

**Manual verification**:
```bash
# Check deployment status
aws ecs describe-services --cluster staging --services api

# Check logs
aws logs tail /aws/ecs/staging-api --follow

# Smoke test
curl https://api-staging.example.com/health
```

### Production Deployment

**Trigger**: Manual (requires approval)

**Steps**:
1. Create deployment tag: `v1.2.3`
2. Approve in GitHub Actions
3. Build production image
4. **Staged rollout**:
   - Deploy to 10% of instances
   - Monitor for 10 minutes
   - If healthy, deploy to 50%
   - Monitor for 10 minutes
   - If healthy, deploy to 100%
5. Run database migrations (if any)
6. Monitor metrics for 30 minutes

**Commands**:
```bash
# Tag release
git tag -a v1.2.3 -m "Release v1.2.3"
git push origin v1.2.3

# Monitor deployment
aws ecs describe-services --cluster production --services api

# Check metrics
open https://grafana.example.com/d/api-dashboard

# Rollback if needed (see Rollback section)
```

### Database Migrations

**Staging**:
```bash
# Automatic via CI/CD after deployment
# Or manual:
aws ecs run-task --cluster staging --task-definition migrate
```

**Production** (requires maintenance window for breaking changes):
```bash
# 1. Create backup
pg_dump -h prod-db.example.com -U admin myapp > backup.sql

# 2. Run migration
aws ecs run-task --cluster production --task-definition migrate

# 3. Verify
psql -h prod-db.example.com -U admin myapp -c "\dt"
```

## Rollback Procedure

### Application Rollback

**Quick rollback** (last known good version):
```bash
# Revert to previous task definition
aws ecs update-service \
  --cluster production \
  --service api \
  --task-definition api:PREVIOUS_VERSION

# Monitor rollback
aws ecs describe-services --cluster production --services api
```

**Full rollback** (specific version):
```bash
# Deploy specific tag
git checkout v1.2.2
# Trigger deployment via CI/CD
```

### Database Rollback

**Non-destructive migrations**: Rollback not needed
**Destructive migrations**: Restore from backup

```bash
# Stop application
aws ecs update-service --cluster production --service api --desired-count 0

# Restore database
pg_restore -h prod-db.example.com -U admin -d myapp backup.sql

# Restart application with old version
aws ecs update-service --cluster production --service api --desired-count 3
```

## Monitoring

### Key Metrics to Watch

**During deployment** (first 30 minutes):
- Error rate (should be < 1%)
- Response time P95 (should be < 500ms)
- Request rate (should match normal traffic)
- Database connection pool (should not be exhausted)

**Dashboards**:
- [Application Dashboard](https://grafana.example.com/d/app)
- [Infrastructure Dashboard](https://grafana.example.com/d/infra)

**Alerts** (Slack #alerts):
- High error rate (> 5%)
- High latency (P95 > 1s)
- Service down
- Database issues

### Health Checks

```bash
# Application health
curl https://api.example.com/health

# Expected response:
# {
#   "status": "healthy",
#   "version": "1.2.3",
#   "database": "connected",
#   "redis": "connected"
# }
```

## Troubleshooting

### Deployment Stuck

**Symptom**: Deployment not progressing
**Check**:
```bash
aws ecs describe-services --cluster production --services api
aws logs tail /aws/ecs/production-api --follow
```
**Fix**: Check task definition, security groups, load balancer

### High Error Rate After Deployment

**Symptom**: Error rate > 5%
**Action**: Immediate rollback
**Investigation**: Check Sentry, application logs

### Database Migration Failed

**Symptom**: Migration task failed
**Action**: Do NOT rollback app, investigate migration
**Fix**: Fix migration, re-run
**Last resort**: Restore from backup

## Emergency Contacts

- **On-call Engineer**: [PagerDuty](https://example.pagerduty.com)
- **DevOps Lead**: Alice (@alice on Slack)
- **CTO**: Bob (bob@example.com)

## Post-Deployment

- [ ] Verify key user flows working
- [ ] Check error tracking (Sentry)
- [ ] Monitor metrics for 1 hour
- [ ] Update CHANGELOG.md
- [ ] Notify team in #engineering
```

## Contributing Guidelines

```markdown
# Contributing to Project

Thank you for your interest in contributing! This document provides guidelines for contributing to this project.

## Code of Conduct

Be respectful, inclusive, and constructive. See [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md).

## Getting Started

1. **Fork the repository**
2. **Clone your fork**: `git clone https://github.com/your-username/project.git`
3. **Create a branch**: `git checkout -b feature/my-feature`
4. **Make changes** (see Development Workflow below)
5. **Test thoroughly**
6. **Submit pull request**

## Development Workflow

### 1. Set Up Environment

```bash
# Install dependencies
npm install

# Copy environment file
cp .env.example .env

# Start services
docker-compose up -d

# Run migrations
npm run migrate
```

### 2. Make Changes

- **Follow style guide** (Prettier, ESLint)
- **Write tests** for new features
- **Update documentation** if needed
- **Commit frequently** with clear messages

### 3. Test Your Changes

```bash
# Run tests
npm test

# Run linter
npm run lint

# Type check
npm run typecheck

# Check coverage
npm run test:coverage
```

### 4. Commit Guidelines

Follow [Conventional Commits](https://www.conventionalcommits.org/):

```
feat: add user authentication
fix: resolve database connection issue
docs: update README with new setup steps
chore: upgrade dependencies
refactor: simplify payment processing
test: add integration tests for API
```

### 5. Pull Request Process

**Before submitting**:
- [ ] Tests pass
- [ ] Code follows style guide
- [ ] Documentation updated
- [ ] No merge conflicts
- [ ] Linked to issue (if applicable)

**PR Template**:
```markdown
## Description
Brief description of changes

## Type of Change
- [ ] Bug fix
- [ ] New feature
- [ ] Breaking change
- [ ] Documentation update

## Testing
How was this tested?

## Checklist
- [ ] Tests pass
- [ ] Code reviewed
- [ ] Documentation updated
```

## Code Standards

### Style Guide

- **TypeScript**: Follow ESLint rules, use Prettier
- **Python**: Follow PEP 8, use Black formatter
- **Naming**: camelCase (JS), snake_case (Python)

### Testing Standards

- **Unit tests**: Test individual functions
- **Integration tests**: Test API endpoints
- **Coverage**: Minimum 80% for new code

### Documentation

- **Functions**: JSDoc or docstrings
- **Complex logic**: Inline comments
- **API changes**: Update OpenAPI spec

## Review Process

1. **Automated checks**: CI/CD runs tests, linting
2. **Code review**: At least one approval required
3. **QA testing**: For significant features
4. **Merge**: Squash and merge to main

## Questions?

- Open an issue
- Ask in #engineering on Slack
- Email: dev@example.com
```

## Generating from Code

**Analyze project structure:**
```bash
# Find key files
ls -la
find . -name "*.md" -o -name "package.json" -o -name "pyproject.toml"

# Analyze dependencies
cat package.json | jq '.dependencies'
cat requirements.txt
```

**Infer tech stack:**
```bash
# Check for frameworks
grep -r "from fastapi" .
grep -r "import React" .

# Database
grep -r "DATABASE_URL" .
```

## Best Practices

✅ **DO:**
- Keep README concise (link to deeper docs)
- Update docs with code changes
- Use diagrams for complex systems
- Provide working examples
- Document the "why" not just "what"
- Include troubleshooting section

❌ **DON'T:**
- Write documentation that duplicates code
- Skip setup instructions
- Assume knowledge
- Let docs get stale
- Forget to document breaking changes

## Constraints

- Must include Quick Start section
- Must list prerequisites
- Must provide examples that work
- Must keep up to date with code
- Must be accessible to new developers
