---
name: ship
description: Invoke the shipper agent to deploy and monitor
---

# Ship Feature

Invoke the shipper agent to deploy code and monitor results.

## Usage

Use this command when you want to:
- Deploy to staging or production
- Set up CI/CD pipelines
- Monitor post-deployment metrics
- Analyze production feedback
- Investigate production issues

## Prerequisites

Before shipping:
- ✅ Code must be built and tested
- ✅ Validation must pass (run `/validate` first)
- ✅ User approval for deployment

## What This Does

The shipper agent will:
1. Verify validation status
2. Set up/use CI/CD pipelines
3. Deploy to target environment
4. Monitor initial health
5. Create deployment report
6. (Optional) Analyze feedback after deployment

## Instructions

Invoke the **shipper** agent with:
- Target environment (staging/production)
- Deployment type (CI/CD setup, manual deploy, rollback)
- Path to validation report (for verification)

## Deployment Flow

### First-time Deployment Setup

**User says:** "Set up deployment for this project"

**You do:**
1. Invoke shipper agent
2. Request: "Set up CI/CD pipeline for this project"
3. Shipper creates:
   - `.github/workflows/deploy.yml` (or equivalent)
   - Deployment scripts
   - Environment configs
4. Shipper documents deployment process

### Deploying a Feature

**User says:** "Deploy the user profile feature"

**You do:**
1. Verify validation passed (check for `docs/validation/user-profile-report.md`)
2. If not validated, warn user and suggest `/validate` first
3. If validated and tests passed:
   - Invoke shipper agent
   - Request: "Deploy user-profile feature to staging"
4. Shipper deploys and monitors
5. Shipper creates deployment report

### Deployment Stages

**Staging deployment:**
```
/ship "Deploy to staging"
→ Shipper deploys to staging
→ Runs smoke tests
→ Reports health status
→ Asks user if ready for production
```

**Production deployment:**
```
/ship "Deploy to production"
→ Shipper verifies staging success
→ Deploys to production
→ Monitors for 30 minutes
→ Creates deployment report
→ Sets up continued monitoring
```

## Post-Deployment Analysis

**After feature has been running:**

**User says:** "Analyze feedback from the profile feature"

**You do:**
1. Invoke shipper agent
2. Request: "Analyze production feedback and metrics for user-profile feature"
3. Shipper analyzes:
   - Error logs
   - Performance metrics
   - User feedback
   - Usage patterns
4. Shipper creates insights report at `docs/feedback/user-profile-insights.md`
5. Shipper recommends next iteration items

## Deployment Report Location

The shipper always creates:
```
docs/deployment/[feature-name]-[environment]-[timestamp].md
```

This includes:
- Deployment summary
- Pre-deployment checks
- Deployment steps taken
- Post-deployment metrics
- Issues detected (if any)
- Next steps

## Safety Checks

The shipper enforces:
- 🚫 Cannot deploy if validation report shows failing tests
- 🚫 Cannot deploy to production without staging verification
- 🚫 Cannot deploy during peak hours (without explicit override)
- ✅ Always creates rollback plan
- ✅ Always monitors post-deployment

## Rollback

**If deployment has issues:**

```
/ship "Rollback user-profile deployment"
→ Shipper executes rollback plan
→ Restores previous version
→ Verifies rollback success
→ Documents incident
```

## Next Steps

**After successful deployment:**
- Monitor for 24-48 hours
- Run `/ship "analyze feedback"` after a few days
- Use insights to plan next iteration with `/plan`

**If deployment fails:**
- Shipper creates incident report
- Run `/build` to fix issues
- Run `/validate` to verify fix
- Run `/ship` again

## Example: Full Deployment Cycle

```
1. /validate
   → Tests pass ✅

2. /ship "deploy to staging"
   → Staging deployed ✅
   → Smoke tests pass ✅

3. User approves production deployment

4. /ship "deploy to production"
   → Production deployed ✅
   → Monitoring shows healthy ✅
   → Deployment report created ✅

5. (Wait 2 days)

6. /ship "analyze feedback"
   → Feedback analyzed ✅
   → Insights report created ✅
   → Next iteration suggestions provided ✅
```
