# Rapid Agile Development Setup

> **TL;DR:** Run `/init-project "your project description"` to set up your system, then use `/rapid-dev "feature description"` to build features. All agents automatically understand your tech stack and requirements. That's it!

This directory contains a complete workflow system for shipping production-ready software at "vibe coding" speeds.

## Quick Start

### Step 1: Initialize Your Project (First Time Only)

Before building features, establish system context:

```bash
/init-project "your project description"
```

This creates:
- `docs/SYSTEM.md` - Tech stack, architecture, requirements (1 page)
- `docs/CONVENTIONS.md` - Coding standards (optional, recommended)
- `docs/architecture/ADR-0001-*.md` - First architectural decision record
- Directory structure for all documentation

**Why?** Agents need to know what tech stack to use, what patterns to follow, and what requirements to meet. CONVENTIONS.md provides opinionated defaults (Conventional Commits, API standards, testing requirements) that ensure consistency. This prevents architectural drift as you build features.

### Step 2: Build Features

Use these commands to run the workflow:

```bash
# Full cycle (plan → build → validate → ship)
/rapid-dev "feature description"

# Individual phases
/plan "feature description"      # Just planning
/build "implement feature"        # Just implementation
/validate "test feature"          # Just testing
/ship "deploy feature"            # Just deployment
```

**All agents automatically read `docs/SYSTEM.md` and `docs/CONVENTIONS.md` before working** to understand your system and follow coding standards.

## How It Works

### System Context (The Foundation)

Before building features, `/init-project` creates:

1. **`docs/SYSTEM.md`** - Single-page system overview
   - Tech stack (what to use)
   - Architecture patterns (how to build)
   - Performance targets (what to achieve)
   - Security requirements (what to enforce)

2. **`docs/CONVENTIONS.md`** - Coding standards (optional, recommended)
   - Git commit format (Conventional Commits)
   - API standards (REST conventions, error formats)
   - Code quality (automated formatting, linting, type safety)
   - Testing requirements (80% coverage, test types)
   - Security standards (auth, rate limiting, secrets management)
   - Pull request template and review process

3. **`docs/architecture/`** - Architecture Decision Records (ADRs)
   - Why we chose each technology
   - Trade-offs accepted
   - Alternatives considered

**Agents read SYSTEM.md and CONVENTIONS.md first** to understand your system and coding standards before planning, building, validating, or deploying.

### Agents (Workflows)

Four specialized agents handle different phases of development:

- **planner** - Translates ideas into structured specs (reads SYSTEM.md for tech stack/patterns)
- **builder** - Implements features from specs (reads SYSTEM.md to know what tech to use)
- **validator** - Tests code and ensures quality (reads SYSTEM.md for quality standards)
- **shipper** - Deploys and monitors production (reads SYSTEM.md for infrastructure)

Each agent:
✅ Reads `docs/SYSTEM.md` FIRST to understand system context
✅ Reads `docs/CONVENTIONS.md` to follow coding standards (if exists)
✅ Reads PHILOSOPHY.md and LOOP-MECHANISM.md for workflow guidance
✅ Creates standardized artifacts (specs, reports)
✅ Follows quality gates consistently
✅ Communicates via structured files

### Skills (Capabilities)

24 specialized skills provide concrete capabilities:

**Infrastructure & DevOps (6)**
- docker-compose-builder, env-config-manager, database-migration-manager
- observability-setup, ci-cd-pipeline-builder, load-test-builder

**Testing (4)**
- tdd-code-generator, integration-test-builder, test-coverage-analyzer, load-test-builder

**Quality & Security (4)**
- modular-code-formatter, security-scanner, performance-profiler, accessibility-auditor

**API & Documentation (4)**
- api-client-generator, openapi-spec-generator, graphql-schema-designer, documentation-generator

**Frontend (2)**
- frontend-component-builder, state-management-architect

**Error Handling & Validation (2)**
- error-handling-standardizer, data-validator-generator

**Git & PR (2)**
- commit-message-formatter, pr-description-generator

**Monitoring (1)**
- feedback-analyzer

Skills are invoked by agents when needed (you don't call them directly).

### Commands

Five slash commands orchestrate the workflow:

- `/rapid-dev` - Full end-to-end cycle
- `/plan` - Just planning phase
- `/build` - Just implementation phase
- `/validate` - Just testing phase
- `/ship` - Just deployment phase

### Coding Conventions (Optional but Recommended)

When you run `/init-project`, you're asked if you want to include opinionated coding conventions. These are **minimal, widely-accepted industry standards** that ensure consistency:

**What's Included:**
- ✅ **Git Commits** - Conventional Commits format (`feat:`, `fix:`, `docs:`, etc.)
- ✅ **API Standards** - REST best practices, standard HTTP codes, error format
- ✅ **Code Quality** - Automated formatting (Prettier, Black), linting, type safety
- ✅ **Testing** - 80% minimum coverage, test types required
- ✅ **Security** - Auth requirements, input validation, rate limiting
- ✅ **Pull Requests** - PR template, merge strategy, review checklist
- ✅ **Performance** - Response time targets, optimization guidelines
- ✅ **Deployment** - Staging → production flow, monitoring requirements

**Why Use Conventions?**
- Agents follow them automatically (consistent code)
- No bikeshedding over tabs vs spaces
- Based on industry best practices (Conventional Commits, etc.)
- Easy to customize or remove sections

**How to Customize:**
1. Edit `docs/CONVENTIONS.md` after creation
2. Remove sections you don't need
3. Adjust standards to match your team
4. Agents will follow your updated conventions

**Example:** If you don't like 80% coverage requirement, change it to 70% in CONVENTIONS.md - agents will follow the updated target.

## The Philosophy

This system balances **speed** (rapid iteration) with **quality** (enterprise reliability).

### We're STRICT About (Non-Negotiable)

**Quality Gates:**
- ✅ Tests must pass before deployment
- ✅ Specs required for production features
- ✅ Validation reports required before shipping
- ✅ Post-deployment monitoring mandatory

**Artifacts:**
- ✅ Spec: `docs/specs/[feature].md`
- ✅ Validation report: `docs/validation/[feature]-report.md`
- ✅ Deployment report: `docs/deployment/[feature]-[env]-[timestamp].md`

**Process:**
- ✅ Builder-validator feedback loop (max 3 iterations)
- ✅ Standard report formats
- ✅ Staged deployments for high-risk features

### We're FLEXIBLE About (Context-Dependent)

**Planning Depth:**
- Critical features → Detailed planning
- Standard features → Moderate planning
- Prototypes → Minimal planning

**Testing Approach:**
- Security-critical → TDD (test-first)
- Standard code → Test-after
- Prototypes → Minimal testing

**Deployment Strategy:**
- High-risk → Staged rollout (5% → 50% → 100%)
- Low-risk → Direct to production
- Experimental → Canary deployment

See `PHILOSOPHY.md` for full details.

## The Feedback Loop

Builder and validator work together in an automatic feedback loop:

```
Iteration 1:
  Validator runs tests → Finds 3 failures
  Validator writes detailed report
  Builder reads report → Fixes issues
  Validator re-tests

Iteration 2 (if needed):
  Validator runs tests → Finds 1 remaining failure
  Validator updates report
  Builder reads report → Fixes last issue
  Validator re-tests → All pass ✅

Deploy!
```

**Max 3 iterations** - if still failing, escalate to user for guidance.

See `LOOP-MECHANISM.md` for full details.

## Directory Structure

```
.claude/
  README.md                  # This file
  PHILOSOPHY.md              # Opinionated workflow philosophy
  LOOP-MECHANISM.md          # Builder-validator feedback loop details

  agents/                    # Workflow agents
    planner.md               # Requirements → specs
    builder.md               # Specs → code
    validator.md             # Code → tested & verified
    shipper.md               # Verified code → deployed

  skills/                    # Specific capabilities (12 total)
    tdd-code-generator/
    modular-code-formatter/
    feedback-analyzer/
    api-client-generator/
    docker-compose-builder/
    env-config-manager/
    test-coverage-analyzer/
    database-migration-manager/
    security-scanner/
    openapi-spec-generator/
    integration-test-builder/
    performance-profiler/

  commands/                  # Slash commands
    init-project.md          # Initialize system (run FIRST!)
    rapid-dev.md             # Full workflow
    plan.md                  # Planning only
    build.md                 # Building only
    validate.md              # Testing only
    ship.md                  # Deployment only

docs/
  SYSTEM.md                  # System overview (created by /init-project)
  templates/                 # Templates for documentation
    SYSTEM.template.md       # Template for SYSTEM.md
    ADR.template.md          # Template for ADRs
  architecture/              # Architecture decisions
    ADR-0001-*.md            # First ADR (tech stack choices)
    README.md                # ADR index
  standards/                 # Coding standards (created as needed)
    README.md                # Standards index
  specs/                     # Feature specs (created by planner)
  validation/                # Test reports (created by validator)
  deployment/                # Deploy reports (created by shipper)
  feedback/                  # Feedback analysis (created by shipper)
```

## Artifacts Created

When you run the workflow, these artifacts are created:

```
docs/
  specs/
    [feature].md                     # Created by planner

  validation/
    [feature]-report.md              # Created by validator

  deployment/
    [feature]-staging-[time].md      # Created by shipper
    [feature]-production-[time].md   # Created by shipper

  feedback/
    [feature]-insights.md            # Created by shipper (post-deploy)
```

These artifacts provide:
- **Traceability** - Understand what was planned, built, tested, deployed
- **Team collaboration** - All teammates can read the same docs
- **Quality assurance** - Evidence of thorough testing
- **Continuous improvement** - Feedback informs next iteration

## Example: Full Feature Development

```bash
# FIRST TIME: Initialize project
/init-project "task management SaaS platform"

# Planner asks questions:
# - Target users? → Small teams (5-50 people)
# - Tech preferences? → Python + FastAPI + PostgreSQL
# - Performance targets? → API < 200ms, 99.9% uptime
# - Security needs? → GDPR compliant, JWT auth

# Creates:
# - docs/SYSTEM.md (tech stack, architecture, requirements)
# - docs/architecture/ADR-0001-initial-tech-stack.md

# NOW ready to build features!

# User runs
/rapid-dev "add user profile feature with avatar upload"

# What happens automatically:

1. Planner agent:
   - Reads docs/SYSTEM.md → Knows to use FastAPI + PostgreSQL
   - Reads PHILOSOPHY.md → Determines planning depth
   - Creates docs/specs/user-profile.md
   - Ensures feature fits system (uses same tech stack)
   - Reports back

2. Builder agent:
   - Reads docs/SYSTEM.md → Uses FastAPI (not Flask/Django)
   - Reads docs/PHILOSOPHY.md and LOOP-MECHANISM.md
   - Reads docs/specs/user-profile.md
   - Implements feature using system's tech stack
   - Applies security requirements from SYSTEM.md
   - Runs basic smoke tests
   - Reports back

3. Validator agent (Iteration 1):
   - Reads docs/SYSTEM.md → Knows performance targets (< 200ms)
   - Reads PHILOSOPHY.md and LOOP-MECHANISM.md
   - Generates tests if missing
   - Runs all tests → 3 failures
   - Checks security requirements from SYSTEM.md
   - Writes docs/validation/user-profile-report.md
   - Reports back: "3 issues found"

4. Builder agent (Fix iteration 1):
   - Reads docs/validation/user-profile-report.md
   - Fixes specific issues
   - Reports back: "Fixed 3 issues"

5. Validator agent (Iteration 2):
   - Re-runs tests → All pass ✅
   - Updates docs/validation/user-profile-report.md
   - Reports back: "Ready for deployment"

6. Shipper agent:
   - Reads docs/SYSTEM.md → Knows infrastructure (AWS ECS)
   - Reads PHILOSOPHY.md
   - Verifies validation passed
   - Deploys to staging → Success
   - Deploys to production using system's infrastructure
   - Creates docs/deployment/user-profile-production-[time].md
   - Monitors metrics from SYSTEM.md targets
   - Reports back: "Deployed successfully, meeting performance targets"

7. (After 2 days)

8. Shipper agent (Feedback analysis):
   - Analyzes production logs
   - Collects metrics and user feedback
   - Creates docs/feedback/user-profile-insights.md
   - Reports back: "Feature performing well, here are improvement suggestions"

Total time: 3-4 hours
Result: Fully deployed, tested, monitored feature that fits the system architecture
```

### Why System Context Matters

**Without SYSTEM.md:**
- Builder might use Flask when project uses FastAPI
- Different auth patterns across features (JWT vs sessions)
- Inconsistent database choices (some use Postgres, some MongoDB)
- No clarity on performance targets
- Architectural drift over time

**With SYSTEM.md:**
- All features use same tech stack ✅
- Consistent patterns across codebase ✅
- Clear quality standards ✅
- Agents make informed decisions ✅
- New developers onboard fast (read 1 page) ✅
```

## Key Benefits

**For Speed:**
- ⚡ Automated workflow - no manual coordination
- ⚡ Feedback loop catches issues automatically
- ⚡ Agents work concurrently where possible
- ⚡ Specs created in minutes, not hours

**For Quality:**
- 🛡️ Strict quality gates enforced automatically
- 🛡️ Comprehensive testing before deployment
- 🛡️ Standardized artifacts for traceability
- 🛡️ Post-deployment monitoring and feedback

**For Teams:**
- 👥 Consistent process across all features
- 👥 Clear artifacts everyone can read
- 👥 New team members onboard quickly
- 👥 Knowledge captured in documents

## Customization

You can customize by editing:

- **PHILOSOPHY.md** - Change when to be strict vs flexible
- **Agent prompts** (agents/*.md) - Adjust agent behavior
- **Skill prompts** (skills/*/SKILL.md) - Modify capabilities
- **Commands** (commands/*.md) - Change workflow orchestration

## Metrics That Matter

Track these to ensure you're balancing speed and quality:

**Speed:**
- Time to first deployment (aim: < 4 hours)
- Iteration cycle time (aim: < 1 day)
- Deployment frequency (aim: multiple per day)

**Quality:**
- Production incident rate (aim: < 1%)
- Time to detect issues (aim: < 5 minutes)
- Time to resolve issues (aim: < 30 minutes)

**Balance:**
- Validation loop iterations (aim: < 2 average)
- Acceptance criteria met (aim: 95%+)
- Post-deploy fixes needed (aim: < 2 per feature)

## Troubleshooting

**Q: Validation loop running 3 times and still failing?**
A: Something fundamental is wrong. The validator will escalate and suggest:
- Manual intervention
- Simplify the spec
- Accept as known limitation

**Q: Agents not reading PHILOSOPHY.md?**
A: Each agent prompt explicitly instructs them to read it first. If they're not, the agent prompt may have been modified.

**Q: Want to skip validation for a quick prototype?**
A: Run `/build` without `/validate`. But remember: no validation = no deployment via `/ship` (quality gate).

**Q: Need different quality standards for different projects?**
A: Copy this `.claude/` directory per project and customize PHILOSOPHY.md for each project's needs.

## Getting Help

1. Read `PHILOSOPHY.md` - Understand the approach
2. Read `LOOP-MECHANISM.md` - Understand feedback loop
3. Look at agent prompts (agents/*.md) - See what each does
4. Try `/rapid-dev` with a small feature - Learn by doing

---

**Remember:** We choose both speed AND reliability. Be strict about outcomes, flexible about methods.
