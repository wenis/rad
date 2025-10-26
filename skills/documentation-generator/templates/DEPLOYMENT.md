# Deployment Runbook Template

Use this template to create comprehensive deployment documentation.

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
\`\`\`bash
# Check deployment status
aws ecs describe-services --cluster staging --services api

# Check logs
aws logs tail /aws/ecs/staging-api --follow

# Smoke test
curl https://api-staging.example.com/health
\`\`\`

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
\`\`\`bash
# Tag release
git tag -a v1.2.3 -m "Release v1.2.3"
git push origin v1.2.3

# Monitor deployment
aws ecs describe-services --cluster production --services api

# Check metrics
open https://grafana.example.com/d/api-dashboard

# Rollback if needed (see Rollback section)
\`\`\`

### Database Migrations

**Staging**:
\`\`\`bash
# Automatic via CI/CD after deployment
# Or manual:
aws ecs run-task --cluster staging --task-definition migrate
\`\`\`

**Production** (requires maintenance window for breaking changes):
\`\`\`bash
# 1. Create backup
pg_dump -h prod-db.example.com -U admin myapp > backup.sql

# 2. Run migration
aws ecs run-task --cluster production --task-definition migrate

# 3. Verify
psql -h prod-db.example.com -U admin myapp -c "\\dt"
\`\`\`

## Rollback Procedure

### Application Rollback

**Quick rollback** (last known good version):
\`\`\`bash
# Revert to previous task definition
aws ecs update-service \\
  --cluster production \\
  --service api \\
  --task-definition api:PREVIOUS_VERSION

# Monitor rollback
aws ecs describe-services --cluster production --services api
\`\`\`

**Full rollback** (specific version):
\`\`\`bash
# Deploy specific tag
git checkout v1.2.2
# Trigger deployment via CI/CD
\`\`\`

### Database Rollback

**Non-destructive migrations**: Rollback not needed
**Destructive migrations**: Restore from backup

\`\`\`bash
# Stop application
aws ecs update-service --cluster production --service api --desired-count 0

# Restore database
pg_restore -h prod-db.example.com -U admin -d myapp backup.sql

# Restart application with old version
aws ecs update-service --cluster production --service api --desired-count 3
\`\`\`

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

\`\`\`bash
# Application health
curl https://api.example.com/health

# Expected response:
# {
#   "status": "healthy",
#   "version": "1.2.3",
#   "database": "connected",
#   "redis": "connected"
# }
\`\`\`

## Troubleshooting

### Deployment Stuck

**Symptom**: Deployment not progressing
**Check**:
\`\`\`bash
aws ecs describe-services --cluster production --services api
aws logs tail /aws/ecs/production-api --follow
\`\`\`
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
