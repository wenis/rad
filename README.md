# Rapid Agile Development (RAD) System

> Ship production-ready software at "vibe coding" speed with AI-powered agents, automated quality gates, and built-in best practices.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

## What is this?

A complete **Claude Code workflow system** that turns feature ideas into production deployments through specialized AI agents with automated testing, security scanning, and deployment monitoring.

**🚀 Speed:** Build and deploy features in hours, not days (2-3x faster with parallel development)
**🛡️ Quality:** Automated testing, security scanning, and validation before every deploy
**📚 Knowledge:** Every decision captured in structured documents (specs, ADRs, reports)
**🤖 Automated:** Four specialized agents (planner, builder, validator, shipper) handle the entire workflow
**⚡ Parallel:** Complex features built with multiple modules simultaneously (v2.0)

## Who is this for?

- **Small teams** (1-10 people) who want to ship fast without sacrificing quality
- **Startups** that need to iterate quickly while maintaining production stability
- **Solo developers** building serious projects who want professional workflows
- **Teams** tired of manual coordination between planning, coding, testing, and deploying

## 🆕 Upgrading from v1.x

If you already have RAD System installed, here's how to upgrade to v2.0 with parallel development support:

### Quick Upgrade (Recommended)

```bash
cd your-project/.claude
git pull origin main
```

That's it! The new parallel workflow is backward compatible - your existing projects will continue to work.

### What's New in v2.0

**🚀 Parallel Development** - Build features 2-3x faster:
- Planner automatically breaks features into parallel modules
- Builder spawns multiple sub-builders simultaneously
- Validators run in parallel for faster feedback
- Real-time progress dashboards show build status

**10 New Skills** for parallel workflow:
- `dependency-analyzer` - Creates optimal build plans
- `module-interface-generator` - Defines clean contracts
- `progress-dashboard-generator` - Real-time build progress
- `integration-conflict-detector` - Catches issues early
- `parallel-test-coordinator` - Faster test execution
- `module-stub-generator` - Unblocks dependencies
- Plus 4 more optimization skills (see Skills section below)

**Enhanced Agents**:
- Planner now creates structured build plans with dependency analysis
- Builder orchestrates parallel workers automatically
- Validator supports module, integration, and system-level testing

### Backward Compatibility

✅ v2.0 is fully backward compatible:
- Sequential builds still work (for simple features)
- Planner auto-detects when to use parallel vs sequential
- Existing slash commands work unchanged
- All existing skills and agents enhanced, not replaced

### No Action Required

Your existing projects work as-is. Parallel development activates automatically for complex features!

---

## Quick Start

> **TL;DR:** Run `/init-project "your project description"` to set up your system, then use `/rapid-dev "feature description"` to build features. All agents automatically understand your tech stack and requirements. That's it!

### Step 1: Install

This is a Claude Code configuration. To use it:

1. Install [Claude Code](https://claude.com/claude-code)
2. In your project directory, create a `.claude` directory and clone this repository into it:
   ```bash
   cd your-project
   mkdir .claude
   cd .claude
   git clone https://github.com/wenis/rad .
   ```

   Or as a one-liner:
   ```bash
   cd your-project && git clone https://github.com/wenis/rad .claude
   ```

### Step 2: Initialize Your Project (First Time Only)

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

### Step 3: Build Features

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

### Keeping Documentation Current

As your project evolves, SYSTEM.md stays up-to-date automatically:

**🤖 Automatic Updates:**
- **Shipper agent** updates SYSTEM.md after successful deployments
  - Detects new tech added (Redis, GraphQL, etc.)
  - Documents new architecture patterns
  - Updates performance metrics

**⚠️ Drift Detection:**
- **Planner agent** warns when specs introduce new tech not in SYSTEM.md
- Example: "This feature needs Redis, but SYSTEM.md doesn't list it - should we add it?"
- Prevents accidental tech sprawl

**🔄 Manual Sync:**
- `/sync-docs` - Scan codebase and sync SYSTEM.md with reality
- Useful after building multiple features or before major releases
- Compares dependencies to documented tech stack
- Presents changes for your approval

**Result:** SYSTEM.md stays accurate without manual maintenance. Agents always work with current information.

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

### Skills (34 Specialized Capabilities)

**🆕 Parallel Development (10 NEW)**
- dependency-analyzer - Analyzes code dependencies for optimal module breakdown
- module-interface-generator - Generates clean contracts for parallel modules
- progress-dashboard-generator - Real-time build progress tracking
- integration-conflict-detector - Detects conflicts before integration
- parallel-test-coordinator - Coordinates parallel test execution
- module-stub-generator - Creates stubs to unblock dependencies
- module-boundary-validator - Validates module isolation
- build-plan-optimizer - Learns from history to optimize strategies
- integration-smoke-test-generator - Fast smoke tests for integration
- parallel-build-metrics-analyzer - Analyzes performance and bottlenecks

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

Seven slash commands orchestrate the workflow:

**Setup & Maintenance:**
- `/init-project` - Initialize project with SYSTEM.md, tech stack, and architecture
- `/sync-docs` - Scan codebase and update SYSTEM.md to match current reality

**Development Workflow:**
- `/rapid-dev` - Full end-to-end cycle (plan → build → validate → ship)
- `/plan` - Just planning phase (or check project status if no feature specified)
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

## Parallel Development (v2.0)

### How It Works

**Automatic parallelization** - Planner detects when features can be built in parallel:

```markdown
Feature: User Authentication

Build Plan (auto-generated):
Phase 1 (Parallel):
  - Module A: Password Validation (3 min)
  - Module B: JWT Tokens (3 min)
  - Module C: Email Service (8 min)

Phase 2 (Parallel - after Phase 1):
  - Module D: Auth API (4 min, depends on A+B)
  - Module E: Password Reset (4 min, depends on A+C)

Integration: Wire modules together (2 min)

Total: ~12 minutes (vs ~25 minutes sequential = 2.1x faster)
```

### Key Features

**Intelligent Planning**:
- `dependency-analyzer` examines code structure
- Identifies truly independent modules
- Creates phases based on dependencies
- Generates clean module interfaces

**Parallel Execution**:
- Builder spawns multiple sub-builders simultaneously
- Each module validates immediately after building
- Progress dashboard shows real-time status
- Fast modules don't wait for slow ones

**Quality Assurance**:
- Module-level validation (tests in isolation)
- Integration validation (tests modules together)
- System-level validation (comprehensive E2E)
- Conflict detection before integration

**Continuous Improvement**:
- Metrics analyzer tracks performance
- Build plan optimizer learns from history
- Identifies bottlenecks automatically
- Suggests optimizations for future builds

### When Parallel Development Activates

**Automatic (default)**:
- Planner assesses feature complexity
- 3+ independent components → Parallel strategy
- 1-2 tightly coupled components → Sequential strategy
- User can override with clarifying questions

**Sequential fallback**:
- Simple features (single responsibility)
- Tightly coupled code (can't be split)
- Prototypes (speed over optimization)
- User preference (asks if unsure)

### Example: Parallel vs Sequential

```bash
# Same command, different execution based on complexity:

/rapid-dev "add simple contact form"
→ Sequential (1 builder, ~8 minutes)

/rapid-dev "add user authentication with email/password"
→ Parallel (3 modules in 2 phases, ~12 minutes vs ~25 sequential)
```

### Benefits

- **2-3x faster builds** for complex features
- **Immediate validation** per module (faster feedback)
- **No blocking** - fast modules complete first
- **Better quality** - isolated testing + integration testing
- **Transparent progress** - see exactly what's happening
- **Continuous learning** - builds get faster over time

---

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
# - docs/CONVENTIONS.md (coding standards)
# - docs/architecture/ADR-0001-initial-tech-stack.md

# NOW ready to build features!

/rapid-dev "add user profile feature with avatar upload"

# What happens automatically:

1. Planner agent:
   - Reads docs/SYSTEM.md → Knows to use FastAPI + PostgreSQL
   - Reads PHILOSOPHY.md → Determines planning depth
   - Creates docs/specs/user-profile.md

2. Builder agent:
   - Reads docs/SYSTEM.md → Uses FastAPI (not Flask/Django)
   - Reads docs/CONVENTIONS.md → Follows API standards
   - Implements feature using system's tech stack

3. Validator agent (Iteration 1):
   - Runs all tests → 3 failures
   - Writes docs/validation/user-profile-report.md

4. Builder agent (Fix iteration 1):
   - Reads validation report → Fixes issues

5. Validator agent (Iteration 2):
   - Re-runs tests → All pass ✅

6. Shipper agent:
   - Deploys to staging → production
   - Monitors metrics
   - Creates deployment report

Total time: 3-4 hours
Result: Fully deployed, tested, monitored feature
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

## Documentation

- **[PHILOSOPHY.md](PHILOSOPHY.md)** - Opinionated workflow philosophy
- **[LOOP-MECHANISM.md](LOOP-MECHANISM.md)** - Builder-validator feedback loop details
- **[agents/](agents/)** - Agent prompts (planner, builder, validator, shipper)
- **[skills/](skills/)** - 24 specialized skill prompts
- **[commands/](commands/)** - Slash command definitions

## Contributing

Contributions welcome! This is an open-source project.

**How to contribute:**

1. **Open an issue** to discuss major changes before implementing
2. **Follow Conventional Commits** for PR titles (`feat:`, `fix:`, `docs:`, etc.)
3. **Test your changes** - Run through a workflow to verify agents work correctly
4. **Update documentation** - Keep README and agent prompts in sync
5. **Add new skills** - Follow the existing skill template structure

**Areas where we'd love help:**
- 🎯 New skills (e.g., mobile app builder, ML model deployer)
- 📚 Better examples and tutorials
- 🐛 Bug fixes and edge case handling
- 🌍 Support for more languages/frameworks
- 📊 Metrics and analytics integration

See [CONTRIBUTING.md](CONTRIBUTING.md) for detailed guidelines.

## License

MIT License - see [LICENSE](LICENSE) for details.

## Credits

Built with ❤️ using [Claude Code](https://claude.com/claude-code)

**Why we built this:** Small teams deserve enterprise-grade workflows without enterprise-grade overhead. This system gives you automated quality gates, comprehensive testing, and deployment monitoring - without the bureaucracy.

---

**Remember:** We choose both speed AND reliability. Be strict about outcomes, flexible about methods.

**Questions?** Open an issue or start with `/init-project` to get going.
