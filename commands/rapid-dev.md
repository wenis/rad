---
name: rapid-dev
description: Full rapid agile development cycle from idea to deployment - orchestrates planner, builder, validator, and shipper agents for complete feature development
---

# Rapid Agile Development Workflow

Execute the complete agile development cycle for a feature, from planning through deployment.

## How This Works

Each agent is programmed to automatically:
- Read `.claude/PHILOSOPHY.md` to understand quality gates and when to be strict vs flexible
- Read `.claude/LOOP-MECHANISM.md` (builder & validator) to understand feedback loop
- Create standardized artifacts (specs, validation reports, deployment reports)
- Follow the defined process without manual intervention

**You don't need to tell agents to read these documents - they do it automatically.**

## Workflow

You will orchestrate the four specialized agents to complete a full development cycle:

1. **Plan** (planner agent) - Creates spec
2. **Build** (builder agent) - Implements feature
3. **Validate** (validator agent) - Tests thoroughly
4. **Ship** (shipper agent) - Deploys and monitors

The builder-validator feedback loop runs automatically (max 3 iterations).

## Instructions

### Phase 1: Requirements & Planning

1. Gather requirements from the user's description
2. Invoke the **planner** agent to:
   - Translate requirements into structured specs
   - Create user stories and acceptance criteria
   - Define test scenarios
   - Identify risks and dependencies
   - Write spec document to `docs/specs/[feature-name].md`

3. Review the spec with the user and get approval before proceeding

### Phase 2: Implementation

4. **Route to appropriate agent based on build strategy:**

   **Read the spec to determine which agent to invoke:**

   a. Read `docs/specs/[feature-name].md`
   b. Find the "Build Plan" or "Execution Strategy" section
   c. Check the strategy:
      - If "Parallel: N modules in M phases" → Invoke **orchestrator** (Step 5a)
      - If "Sequential: Single builder" → Invoke **builder** (Step 5b)
      - If unclear → Default to builder (Step 5b)

5a. **If Parallel Strategy - Invoke Orchestrator:**

    **Use this EXACT minimal prompt:**
    ```
    Orchestrate the build for docs/specs/[feature-name].md
    ```

    **That's it. Nothing else.**

    The orchestrator will:
    - Parse the build plan
    - Spawn multiple builder agents in parallel
    - Monitor their progress
    - Coordinate validation loops
    - Handle integration
    - Report final status

    **Skip to step 6** (orchestrator handles everything)

5b. **If Sequential Strategy - Invoke Builder:**

    **Use this EXACT minimal prompt:**
    ```
    Read the spec at docs/specs/[feature-name].md and implement the feature.
    ```

    **That's it. Nothing else.**

    The builder will:
    - Read the spec
    - Implement the feature
    - Run basic smoke tests
    - Report what was built

    **Continue to step 6**

**🚨 ENFORCEMENT CHECKLIST - Before Invoking Agent:**
- [ ] Prompt is < 20 words
- [ ] Prompt is EITHER: "Orchestrate..." OR "Read the spec...and implement..."
- [ ] NO additional context, guidelines, or instructions
- [ ] NO preambles, requirements, or numbered steps

**If your prompt fails ANY check, DELETE IT and use the exact template above.**

6. **Review the code/status with user if needed**

### Phase 3: Testing & Validation (with Feedback Loop)

7. Invoke the **validator** agent to:
   - Generate comprehensive tests based on the spec
   - Run all tests (unit, integration, edge cases)
   - Check for security issues
   - Verify acceptance criteria are met
   - Create validation report at `docs/validation/[feature-name]-report.md`

8. **Validation Feedback Loop** (max 3 iterations):

   **Note:** For parallel builds, the orchestrator handles validation loops for each module. For sequential builds, you manage the feedback loop.

   **Iteration 1:**
   - If validation passes → proceed to Phase 4
   - If validation fails:
     - Validator writes detailed report with specific failures
     - Invoke **builder** agent: "Read validation report at docs/validation/[feature-name]-report.md and fix all issues"
     - Builder reads report, fixes issues, confirms completion
     - Invoke **validator** again

   **Iteration 2 (if still failing):**
   - Validator runs tests again
   - If passes → proceed to Phase 4
   - If fails:
     - Validator updates report with new/remaining issues
     - Invoke **builder** again with updated report
     - Builder fixes, confirms completion
     - Invoke **validator** again

   **Iteration 3 (if still failing):**
   - Validator runs final test
   - If passes → proceed to Phase 4
   - If fails:
     - STOP the loop
     - Report to user: "After 3 iterations, tests still failing"
     - Show summary of remaining issues
     - Ask user for guidance:
       - Option A: Continue with manual fixes
       - Option B: Revisit spec (may be too ambitious)
       - Option C: Accept current state and document known issues

9. Only proceed to deployment if validation passes

### Phase 4: Deployment

10. Ask user if they want to deploy:
    - If yes, continue to step 11
    - If no, end workflow and provide summary

11. Invoke the **shipper** agent to:
    - Verify all tests are passing
    - Set up CI/CD if needed
    - Deploy to staging first
    - Run smoke tests in staging
    - Deploy to production (with user approval)
    - Monitor initial metrics
    - Create deployment report

12. After deployment, invoke **shipper** again to:
    - Analyze post-deployment feedback
    - Generate insights report
    - Identify any issues or improvements
    - Suggest next iteration items

### Phase 5: Iteration Planning

13. Provide complete summary to user:
    - What was built
    - Test results
    - Deployment status
    - Feedback insights
    - Recommended next steps

## Example Execution

**User request:** "I want to add user authentication with email and password"

**Phase 1: Planning**
```
Invoking planner agent...

Planner creates:
- docs/specs/user-authentication.md with:
  - User stories
  - Acceptance criteria (12 char passwords, rate limiting, etc.)
  - Test scenarios
  - Security considerations
  - Next steps
```

**Phase 2: Building**
```
Invoking builder agent with:
"Read the spec at docs/specs/user-authentication.md and implement the feature."

Builder announces:
📋 BUILD PLAN DETECTED
Strategy: Sequential (single builder)
Mode: Direct Build

Builder implements:
- src/auth/password.py (validation, hashing)
- src/auth/jwt.py (token generation)
- src/api/auth.py (login endpoint)
- Basic smoke tests pass
```

**Phase 3: Validation**
```
Invoking validator agent...

Validator results:
- Generated 23 tests
- 22 passed, 1 failed (weak password test)

Invoking builder agent to fix...

Validator results (retry):
- 23 tests all passing
- Security checks pass
- Coverage: 94%
- Ready for deployment
```

**Phase 4: Deployment**
```
User approves deployment.

Invoking shipper agent...

Shipper deploys:
- Staging: ✅ Deployed, smoke tests pass
- Production: ✅ Deployed
- Monitoring: Error rate 0.1%, response time 95ms
- Deployment report saved
```

**Phase 5: Post-deployment**
```
Invoking shipper for feedback analysis...

Insights:
- 50 successful logins in first hour
- 3 failed login attempts (expected)
- No errors detected
- User feedback: Positive

Recommended next iteration:
- Add OAuth (Google, GitHub)
- Add password reset flow
- Add 2FA option
```

## Flexibility

This workflow can be adapted based on the situation:

### Quick prototype mode
- Skip validator for initial prototype
- Get user feedback early
- Add tests later

### Critical feature mode
- Extra validation phase
- Security review
- Staged rollout (5% → 50% → 100%)

### Bug fix mode
- Skip planner (spec already exists)
- Go straight to builder
- Validate fix
- Deploy hotfix

## Available Skills

The agents can leverage these skills when needed:
- `tdd-code-generator` - For test-driven development approach
- `modular-code-formatter` - For code cleanup after rapid prototyping
- `api-client-generator` - When integrating external APIs
- `docker-compose-builder` - For containerization
- `env-config-manager` - For environment setup
- `test-coverage-analyzer` - For coverage analysis
- `feedback-analyzer` - For post-deployment analysis

## Success Criteria

A successful rapid development cycle includes:

✅ Clear spec document created and approved
✅ Feature implemented according to spec
✅ All tests passing (unit, integration, edge cases)
✅ No critical security issues
✅ Successfully deployed to production
✅ Monitoring shows healthy metrics
✅ Feedback analyzed and next steps identified

## Anti-patterns to Avoid

❌ Starting to code without a spec
❌ Skipping validation before deployment
❌ Deploying with failing tests
❌ Not monitoring after deployment
❌ Ignoring user feedback

## Execution Guidelines

- **Move fast**: Don't over-plan, get to working code quickly
- **Stay flexible**: Adapt workflow based on user feedback
- **Communicate clearly**: Keep user informed at each phase
- **Fail fast**: Catch issues early in validation
- **Learn and iterate**: Use feedback to improve next cycle

## Output Artifacts

At the end of the workflow, you should have created:

1. `docs/specs/[feature-name].md` - Feature specification
2. Source code files for the feature
3. Test files with comprehensive coverage
4. `docs/validation/[feature-name]-report.md` - Validation report
5. `docs/deployment/[feature-name]-deployment.md` - Deployment report
6. `docs/feedback/[feature-name]-insights.md` - Feedback analysis

## Notes

- User can stop the workflow at any phase
- User can request changes and restart from any phase
- Agents work autonomously but report back to main conversation
- Each agent has its own context, so pass relevant info between agents
- Save all reports to files for future reference
