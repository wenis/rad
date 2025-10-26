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

### Keeping Documentation Current

Documentation that drifts from reality is worse than no documentation - it misleads. RAD keeps SYSTEM.md current automatically:

**Automatic Updates (Shipper):**
- After successful production deployments, shipper scans what was built
- Detects new tech added (Redis, GraphQL, new libraries)
- Detects new patterns (event-driven, caching layers, WebSockets)
- Updates `docs/SYSTEM.md` Tech Stack and Architecture sections
- Recommends ADRs for major architectural changes

**Drift Detection (Planner):**
- When planning features, planner checks if spec introduces new tech
- Warns: "This needs Redis, but SYSTEM.md doesn't list it - should we add it?"
- Prevents accidental tech sprawl and ensures intentional decisions
- When resuming work (/plan with no feature), planner scans dependencies
- Reports if SYSTEM.md seems outdated, suggests `/sync-docs`

**Manual Sync (/sync-docs):**
- Scans dependency files (package.json, requirements.txt, etc.)
- Compares actual tech stack to documented stack
- Presents findings for user approval
- Updates SYSTEM.md to match reality
- Useful after building many features or before major releases

**Result:** SYSTEM.md stays accurate. Agents always work with current information. New team members see the real system, not historical fiction.

## Core Tension

**The Paradox:** We want to move fast like a startup prototype, but ship with the reliability of enterprise software.

Traditional approaches fail:
- **Pure "vibe coding"** (LLM-generated code, no process) → Fast but unreliable
- **Enterprise waterfall** (extensive planning, reviews, testing) → Reliable but slow

**Our solution:** Be rigid about OUTCOMES, flexible about METHODS.

---

## Agent Decision-Making Principles

To achieve rapid + reliable development, agents must make high-quality decisions quickly. These principles ensure decision quality:

### 1. Think Before Acting (Always)

**For complex decisions, agents must output their reasoning explicitly.**

When deciding:
- Planning depth (light vs heavy)
- Build strategy (parallel vs sequential)
- Deployment strategy (staged vs direct)
- Test approach (TDD vs test-after)
- Issue severity (critical vs warning)

**Agents use structured reasoning:**

<thinking>
1. What information do I have? [Known facts]
2. What are my options? [Alternatives]
3. What are the tradeoffs? [Pros/cons]
4. What is my recommendation and why? [Decision + rationale]
</thinking>

**Why this matters:**
- Visible reasoning catches flawed logic before action
- Decisions are auditable and improvable
- User can intervene if reasoning is wrong
- Pattern emerges for future decisions

**Rule:** Complex decisions without visible reasoning often fail. Always show your work.

### 2. Define Success Upfront (Always)

**Before starting work, agents establish clear success criteria.**

**Planner:** What makes a spec ready for builder?
- All acceptance criteria measurable
- Build plan clearly sequential or parallel
- Tech stack alignment verified
- No ambiguous requirements remain

**Builder:** What makes code ready for validation?
- All spec files created/modified
- Compiles/runs without errors
- Basic smoke test passes
- Follows tech stack from SYSTEM.md

**Validator:** What makes code ready for deployment?
- All critical tests pass
- Security requirements met
- Performance targets achieved
- No blocking issues remain

**Shipper:** What makes deployment successful?
- Staging tests pass
- Health checks pass at each rollout stage
- Metrics meet or exceed targets
- SYSTEM.md updated if needed

**Why this matters:**
- Prevents "definition drift" (unclear when done)
- Enables objective quality assessment
- Reduces rework from missed requirements

**Rule:** If success criteria aren't clear upfront, work is likely to fail validation.

### 3. Self-Check Before Delivery (Always)

**Agents validate their own work before handing off.**

**Self-check questions:**

Planner:
- "Could a developer implement this spec without asking questions?"
- If NO → spec is incomplete

Builder:
- "Could the validator run tests against this code right now?"
- If NO → build is incomplete

Validator:
- "Would I deploy this code to production right now?"
- If NO → identify blocking issues as critical

Shipper:
- "Would I be comfortable going on vacation after this deployment?"
- If NO → something is wrong, investigate

**Why this matters:**
- Catches issues before they reach next agent
- Reduces validation loop iterations
- Faster overall cycle time

**Rule:** Self-check questions reveal gaps that cause rework. Use them liberally.

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

## Parallel Development (v2.0)

### What Changed

v2.0 introduces **parallel module development** - building independent parts of a feature simultaneously for 2-3x speed improvement.

**Key principle:** Maintain all quality gates while enabling parallel execution.

### Parallel Module Design

**When planner creates parallel modules:**

✅ **DO:**
- Design modules with clear boundaries (single responsibility)
- Define interfaces before building (contracts)
- Ensure modules are independently testable
- Identify dependencies explicitly (phases)
- Use dependency injection (not global state)

❌ **DON'T:**
- Create circular dependencies (blocks parallelization)
- Share mutable global state (race conditions)
- Tightly couple modules (reduces benefit)
- Make modules too granular (overhead > benefit)

### Parallel Execution Strategy

**Planner's decision framework:**

**Use Parallel When:**
- Feature has 3+ independent components
- Components can be built without waiting for each other
- Complexity justifies coordination overhead
- Team/timeline benefits from speed boost

**Use Sequential When:**
- Simple feature (1-2 components)
- Tightly coupled components
- Prototype/experiment (speed over optimization)
- User explicitly prefers simple approach

**Let planner decide** - It analyzes dependencies automatically and chooses optimal strategy.

### Module Isolation Requirements

**For parallel modules to work:**

1. **No circular dependencies**
   - Module A can depend on Module B
   - Module B cannot depend on Module A
   - Dependencies must form a directed acyclic graph (DAG)

2. **No shared mutable state**
   - No global variables modified by multiple modules
   - Use dependency injection instead
   - Pass state as parameters

3. **Clear interfaces**
   - Define contracts before building
   - All exports explicitly declared
   - Type-safe interactions

4. **Independent testing**
   - Module tests run in isolation
   - Mock external dependencies
   - No cross-module test dependencies

### Validation Strategy

**Three-tier validation:**

**Tier 1: Module-Level** (parallel)
- Each module validated immediately after building
- Tests run in isolation (mocked dependencies)
- Fast feedback (don't wait for other modules)
- 3 iterations max per module

**Tier 2: Integration-Level** (after modules pass)
- Test modules working together
- Verify integration points
- Catch cross-module issues
- 3 iterations max for integration

**Tier 3: System-Level** (comprehensive)
- End-to-end acceptance criteria
- Full security and performance checks
- Complete user flows
- Final quality gate before deployment

### Integration Principles

**After parallel modules complete:**

1. **Detect conflicts first** - Run integration-conflict-detector before wiring
2. **Smoke test fast** - Run quick integration tests (30s) before full validation
3. **Fix at source** - Send issues back to module builders, not integration builder
4. **Keep integration thin** - Glue code only, not business logic

### Speed vs. Quality in Parallel

**Strict (non-negotiable):**
- ✅ All modules must pass validation before integration
- ✅ Integration tests must pass before deployment
- ✅ No module bypasses quality gates
- ✅ Same 3-iteration limit per module

**Flexible (context-dependent):**
- Module complexity (some simple, some complex)
- Validation thoroughness (critical vs standard modules)
- Integration strategy (stub dependencies vs wait)

**Quality gates remain the same** - parallel execution speeds up the process without reducing quality.

### Continuous Improvement

**Learn from each parallel build:**

1. **Track metrics** - Build time, speedup, bottlenecks
2. **Identify patterns** - Which module types are slow? Which fail often?
3. **Optimize strategies** - Use historical data for better planning
4. **Improve estimates** - Learn actual vs estimated durations

**build-plan-optimizer** learns over time to suggest better module breakdowns.

### Anti-Patterns for Parallel Development

### ❌ Too Many Modules

**Symptoms:**
- Breaking feature into 10+ tiny modules
- Coordination overhead > time savings
- More time integrating than building

**Impact:** Slower than sequential despite parallelization

**Fix:** Aim for 2-5 modules per phase (sweet spot)

### ❌ Premature Parallelization

**Symptoms:**
- Using parallel for simple 2-component feature
- Overhead not justified by complexity
- Sequential would be faster

**Impact:** Wasted time on coordination

**Fix:** Let planner decide - it knows when parallel helps

### ❌ Hidden Dependencies

**Symptoms:**
- Modules marked "independent" but actually depend on each other
- Circular imports discovered during integration
- Global state causing race conditions

**Impact:** Integration failures, wasted parallel build time

**Fix:** Run module-boundary-validator before starting build

### ❌ Integration Hell

**Symptoms:**
- All modules pass individual validation
- Integration phase fails repeatedly
- Type mismatches, missing exports, incompatible interfaces

**Impact:** Lose all time savings in integration debugging

**Fix:** Generate interfaces upfront, run conflict detector early

### ✅ Right Balance

**Characteristics:**
- 2-5 modules per phase
- Clear interfaces defined before building
- Modules truly independent (no hidden deps)
- Fast modules don't block slow ones
- Immediate validation per module
- Clean integration with smoke tests

**Impact:** 2-3x faster builds with same quality

---

## Summary: The Rules

### Strict Rules (Always Follow)

1. **Establish system context** - Run `/init-project` before first feature, maintain SYSTEM.md
2. **Create artifacts** - Spec, validation report, deployment report
3. **Pass quality gates** - Tests pass, no critical issues, acceptance criteria met
4. **Use feedback loop** - Max 3 validator-builder iterations (per module in parallel builds)
5. **Follow standard formats** - Consistent templates for all artifacts
6. **Monitor production** - Always measure impact of deployments
7. **🆕 Maintain module isolation** - No circular deps, no shared mutable state (parallel builds)

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
