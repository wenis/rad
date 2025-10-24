# Rapid Agile Development Philosophy

**Goal:** Enable small teams to ship production-ready enterprise software at "vibe coding" speeds without sacrificing quality.

This is the guiding philosophy for how the agents, skills, and workflows should operate. It balances **speed** (rapid iteration) with **quality** (enterprise-grade output).

## System Context: The Foundation

**BEFORE** building features, establish system context with `/init-project`.

This creates:
- **`docs/SYSTEM.md`** - Single-page overview (tech stack, architecture, requirements)
- **`docs/architecture/`** - Architecture Decision Records (ADRs) for major choices
- **`docs/standards/`** - Coding conventions (created as needed)

**Why this matters:**
- Agents know what tech stack to use (don't guess!)
- Consistency across features (same patterns, same tools)
- Architectural decisions are documented (know the "why")
- New team members onboard quickly (read SYSTEM.md → understand project)

**All agents read SYSTEM.md FIRST** before working on any feature.

## Core Tension

**The Paradox:** We want to move fast like a startup prototype, but ship with the reliability of enterprise software.

Traditional approaches fail:
- **Pure "vibe coding"** (LLM-generated code, no process) → Fast but unreliable
- **Enterprise waterfall** (extensive planning, reviews, testing) → Reliable but slow

**Our solution:** Be rigid about OUTCOMES, flexible about METHODS.

---

## What We're Opinionated About (Non-Negotiable)

These are the QUALITY GATES - hard requirements for enterprise software:

### 1. Artifact Creation (Always Required)

Every feature must produce these artifacts:

✅ **Specification document** (`docs/specs/[feature].md`)
- Written by planner agent
- Contains user stories, acceptance criteria, test scenarios
- Serves as source of truth

✅ **Validation report** (`docs/validation/[feature]-report.md`)
- Written by validator agent
- Shows test results, issues found, coverage
- Required before deployment

✅ **Deployment report** (`docs/deployment/[feature]-[env]-[timestamp].md`)
- Written by shipper agent
- Documents deployment steps, metrics, health
- Required for production deployments

❌ **Never deploy without these artifacts** - they provide traceability and enable team collaboration.

### 2. Quality Gates (Always Enforced)

These gates MUST pass before proceeding:

**Before Building:**
- ✅ Spec must exist and be approved (or explicitly skip for prototypes)

**Before Deployment:**
- ✅ All critical tests must pass
- ✅ No security vulnerabilities in validator report
- ✅ Acceptance criteria met (if spec exists)
- ✅ Code review completed (for production)

**During Deployment:**
- ✅ Staging deployment must succeed before production
- ✅ Health checks must pass post-deployment
- ✅ Rollback plan must exist

❌ **Never compromise on these gates** - they prevent production incidents.

### 3. Feedback Loop (Always Enabled)

The builder-validator loop is mandatory for quality:

**Validation Loop Rules:**
1. Validator MUST create detailed report when tests fail
2. Builder MUST read the report and fix specific issues
3. Loop runs max 3 iterations before escalating to user
4. Each iteration must make measurable progress

**Why 3 iterations?**
- Iteration 1: Fix obvious issues
- Iteration 2: Fix edge cases
- Iteration 3: Final cleanup
- If still failing → fundamental problem, needs human insight

❌ **Never skip validation** - catching issues early is 10x cheaper than fixing in production.

### 4. Standard Formats (Always Consistent)

All artifacts follow standard templates:

**System context:**
- `docs/SYSTEM.md` - One-page overview (tech stack, targets, requirements)
- `docs/architecture/ADR-XXXX-*.md` - Architecture Decision Records

**Feature artifacts:**
- **Specs:** User stories, acceptance criteria, test scenarios, risks
- **Validation reports:** Summary, test results, issues by priority, recommendations
- **Deployment reports:** Pre-checks, steps, metrics, post-deployment health

**Why consistent formats?**
- Team members can quickly scan any artifact
- Automations can parse and react to reports
- Knowledge transfers easily across features
- New team members onboard faster
- Agents understand system context before building

❌ **Never invent new formats** - consistency is more valuable than perfection.

### 5. Monitoring & Feedback (Always Measured)

Every production deployment must be monitored:

**Required metrics:**
- Error rate (compared to baseline)
- Response time (p95, p99)
- User impact (affected users, requests)

**Required actions:**
- Monitor for 30 minutes post-deployment
- Create feedback analysis within 48 hours
- Identify issues for next iteration

❌ **Never "fire and forget"** - production is where you learn the most.

---

## What We're Flexible About (Adapt to Context)

These are the METHODS - adapt based on situation:

### 1. Planning Depth (Context-Dependent)

**Heavy planning when:**
- Security-critical feature (auth, payment, data access)
- Multi-team coordination required
- Complex business logic
- High user impact

→ Detailed spec with edge cases, security review, multiple stakeholders

**Light planning when:**
- Internal tool
- Low-risk UI change
- Quick experiment
- Bug fix

→ Brief spec, focus on happy path, fast iteration

**No planning when:**
- Throwaway prototype
- Proof of concept
- Testing a hypothesis
- Solo developer exploration

→ Skip planner, go straight to building, add spec later if kept

### 2. Testing Approach (Risk-Based)

**TDD (test-first) when:**
- Security-critical code (auth, encryption, payments)
- Complex algorithms
- Mission-critical business logic
- High-risk changes

→ Use `tdd-code-generator` skill, Red-Green-Refactor

**Test-after when:**
- Standard CRUD operations
- UI components
- Integration with well-tested libraries
- Low-risk utilities

→ Build first, validator generates tests

**Minimal testing when:**
- Internal scripts
- Prototypes
- One-time migrations
- Configuration changes

→ Smoke tests only, manual verification

**Coverage targets (flexible):**
- Critical paths: 95%+
- Core business logic: 85%+
- API endpoints: 80%+
- Utilities: 75%+
- UI components: 70%+
- Prototypes: 60%+

### 3. Implementation Speed (Phase-Based)

**Two-phase approach:**

**Phase 1: Prototype (Speed first)**
- Get something working quickly
- Hardcode values if needed
- Focus on happy path
- Minimal error handling
- Goal: Demonstrate feasibility

**Phase 2: Production (Quality second)**
- Refactor for production
- Add error handling
- Make configurable
- Add logging/monitoring
- Optimize performance

**When to use single phase:**
- Simple, well-understood features → Skip prototype, build production-ready
- Experimental features → Stay in prototype until validated
- Mission-critical → Build production-ready from start

### 4. Code Style & Structure (Tool-Enforced)

Don't argue about style - use tools:

**Formatting:** Let `modular-code-formatter` handle it
**Linting:** Run automated linters (eslint, pylint, gofmt)
**Type checking:** Run but don't block on it initially

**When to be strict:**
- Production code: Full linting, type checking
- Prototypes: Skip, move fast

### 5. Deployment Strategy (Risk-Based)

**Staged rollout when:**
- User-facing changes
- High-risk features
- Large refactors
- Database migrations

→ Deploy 5% → 50% → 100% with monitoring

**Direct to production when:**
- Bug fixes (verified in staging)
- Internal tools
- Low-risk changes
- Configuration updates

→ Deploy 100%, monitor closely

**Canary deployments when:**
- Major version upgrades
- Infrastructure changes
- Unproven features

→ Run old and new in parallel, compare metrics

---

## Decision Framework

When unsure whether to be strict or flexible, ask:

### Is this about OUTCOME or METHOD?

**Outcome → Be strict:**
- "Must all tests pass?" → YES, always
- "Must we create a spec?" → YES, for production features
- "Must we monitor after deploy?" → YES, always

**Method → Be flexible:**
- "Should we use TDD?" → Depends on criticality
- "How detailed should the spec be?" → Depends on complexity
- "What test coverage target?" → Depends on code type

### What's the risk if we skip this?

**High risk → Be strict:**
- Security vulnerability
- Data loss
- Financial impact
- Legal/compliance issues

**Low risk → Be flexible:**
- UI aesthetics
- Performance (within reason)
- Code organization
- Documentation details

### Can we iterate and improve later?

**Hard to fix later → Be strict:**
- Database schema design
- API contracts (external)
- Security architecture
- Deployment pipelines

**Easy to improve later → Be flexible:**
- Code organization
- Variable naming
- Comment quality
- Test coverage (can add more)

---

## Balancing Speed vs. Quality

### When to prioritize SPEED

- Early stage projects (finding product-market fit)
- Internal tools (smaller user impact)
- Prototypes and experiments
- Low-risk features
- Fast feedback needed

**How:**
- Light specs (bullet points)
- Test after building
- Skip TDD
- Single-phase implementation
- Deploy to staging only initially

### When to prioritize QUALITY

- Production features (user-facing)
- Security-critical code
- Financial transactions
- Large user base
- High cost of failure

**How:**
- Detailed specs
- TDD approach
- Multiple validation iterations
- Staged rollouts
- Comprehensive monitoring

### The Sweet Spot

Most features should be in the middle:

1. **Spec in 15 minutes** (not 2 hours) - focus on key requirements
2. **Build working code fast** (prototype) - demonstrate feasibility
3. **Refine to production quality** - add error handling, tests
4. **Validate thoroughly** - run the full test suite
5. **Deploy to staging first** - verify in realistic environment
6. **Monitor production** - catch issues early

**Time breakdown for typical feature:**
- Planning: 20% (quick spec)
- Building: 40% (fast prototype + refinement)
- Testing: 25% (thorough validation)
- Deployment: 10% (staged rollout)
- Monitoring: 5% (post-deploy analysis)

---

## Anti-Patterns to Avoid

### ❌ Too Rigid (Enterprise Waterfall)

**Symptoms:**
- Spec takes 3 days for simple feature
- Every line of code requires review
- 10 approval gates before deployment
- Perfect test coverage demanded
- Zero tolerance for any issues

**Impact:** Slow to market, team frustration, missed opportunities

### ❌ Too Loose (Cowboy Coding)

**Symptoms:**
- No specs, just start coding
- Tests written after deployment
- "Works on my machine" deploys
- No monitoring or feedback
- Fix issues reactively in production

**Impact:** Production incidents, technical debt, user trust erosion

### ✅ Right Balance (Rapid Agile)

**Characteristics:**
- Spec in minutes, not days
- Quality gates are strict but quick
- Tests written during/after building
- Deploy frequently with monitoring
- Fix issues in feedback loop

**Impact:** Fast iteration, reliable software, happy users

---

## Metrics That Matter

Track these to ensure balance:

### Speed Metrics
- **Time to first deployment:** Spec → Staging (aim: < 4 hours)
- **Iteration cycle time:** Spec → Production (aim: < 1 day)
- **Deployment frequency:** How often we ship (aim: multiple per day)

### Quality Metrics
- **Production incident rate:** Errors per deployment (aim: < 1%)
- **Time to detect issues:** Deploy → Detection (aim: < 5 minutes)
- **Time to resolve issues:** Detection → Fix deployed (aim: < 30 minutes)

### Balance Metrics
- **Validation loop iterations:** Average per feature (aim: < 2)
- **Spec-to-code alignment:** Acceptance criteria met (aim: 95%+)
- **Post-deploy changes:** Fixes needed after deploy (aim: < 2)

**The sweet spot:**
- Fast to deploy AND low incident rate
- Quick iterations AND high acceptance criteria met
- Minimal post-deploy fixes AND frequent deployments

---

## Team Philosophy

### For Developers

**Move fast, but:**
- Write specs (even brief ones)
- Run tests before deploying
- Monitor after deploying
- Learn from production

**Don't stress about:**
- Perfect code on first try
- 100% test coverage
- Beautiful abstractions initially
- Comprehensive documentation

**Iterate:**
- Ship working code quickly
- Gather feedback
- Refine based on real usage
- Improve over time

### For Product/Business

**Expect:**
- Fast iterations (hours, not weeks)
- Reliable deployments (low incident rate)
- Continuous improvement (feedback-driven)
- Transparent process (artifacts for everything)

**Don't expect:**
- Perfect features on first release
- Zero bugs ever
- Instant complex features
- No tradeoffs

**Trust the process:**
- Specs ensure we build the right thing
- Validation ensures quality
- Monitoring catches issues fast
- Feedback drives improvement

---

## Summary: The Rules

### Strict Rules (Always Follow)

1. **Establish system context** - Run `/init-project` before first feature, maintain SYSTEM.md
2. **Create artifacts** - Spec, validation report, deployment report
3. **Pass quality gates** - Tests pass, no critical issues, acceptance criteria met
4. **Use feedback loop** - Max 3 validator-builder iterations
5. **Follow standard formats** - Consistent templates for all artifacts
6. **Monitor production** - Always measure impact of deployments

### Flexible Rules (Adapt to Context)

1. **Planning depth** - Light to heavy based on risk
2. **Testing approach** - TDD vs test-after based on criticality
3. **Implementation speed** - Prototype vs production-ready based on phase
4. **Code style** - Let tools handle it
5. **Deployment strategy** - Staged vs direct based on risk

### Golden Rule

**When in doubt, ask:**

"What's the fastest way to ship this reliably?"

- If you can skip something AND still be reliable → skip it
- If you can't guarantee reliability without it → do it

Speed without reliability = chaos
Reliability without speed = irrelevant

**We choose both.**
