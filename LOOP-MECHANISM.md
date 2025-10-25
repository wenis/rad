# Builder-Validator Feedback Loop

This document explains how the builder and validator agents communicate and iterate to ensure quality.

## The Challenge

Agents run in separate contexts and can't directly communicate. We need a mechanism for:
1. Validator to tell builder what's broken
2. Builder to understand and fix specific issues
3. Multiple iterations without manual intervention
4. Automatic termination to prevent infinite loops

## The Solution: File-Based Communication

Agents communicate through structured report files:

```
Validator → writes report → Builder reads report → fixes issues → Validator re-tests
```

## Detailed Flow

### Iteration 1: Initial Validation

**Step 1:** Validator runs tests
```bash
Validator agent invoked: "Test the user-profile feature against docs/specs/user-profile.md"

Validator actions:
1. Reads spec from docs/specs/user-profile.md
2. Finds existing tests or generates new ones
3. Runs all tests (pytest/jest/etc)
4. Analyzes results
5. Checks for security issues
6. Writes detailed report to docs/validation/user-profile-report.md
```

**Step 2:** Validator report example
```markdown
# Validation Report - User Profile

## Summary
- Total Tests: 15
- Passed: 12
- Failed: 3
- Coverage: 78%

## Failed Tests

### Test: test_update_email_validation
**File:** tests/test_profile.py:45
**Status:** FAILED
**Error:**
```
AssertionError: Expected 400 status code for invalid email, got 200
```
**Issue:** Email validation not working - accepts invalid email "notanemail"
**Fix:** Add email format validation in api/profile.py:validateEmail()

### Test: test_avatar_upload_size_limit
**File:** tests/test_profile.py:67
**Status:** FAILED
**Error:**
```
AssertionError: Expected 413 status code for large file, got 500
```
**Issue:** No file size check before upload - should reject files > 5MB
**Fix:** Add file size validation in api/profile.py:uploadAvatar()

### Test: test_concurrent_updates
**File:** tests/test_profile.py:89
**Status:** FAILED
**Error:**
```
DatabaseError: Duplicate key violation
```
**Issue:** Race condition when two updates happen simultaneously
**Fix:** Add optimistic locking or transaction handling in api/profile.py:updateProfile()

## Recommendations
1. Add email validation regex
2. Check file size before processing upload
3. Use database transactions or version field for concurrent updates
```

**Step 3:** Builder reads report and fixes
```bash
Builder agent invoked: "Read the validation report at docs/validation/user-profile-report.md and fix all issues"

Builder actions:
1. Reads docs/validation/user-profile-report.md
2. Identifies 3 specific failures
3. For each failure:
   - Reads the relevant code file
   - Understands the issue from the report
   - Implements the fix
   - Verifies locally if possible
4. Reports back: "Fixed 3 issues: email validation, file size check, concurrent update handling"
```

**Step 4:** Re-validate
```bash
Validator agent invoked: "Re-test user-profile feature"

Validator actions:
1. Runs all tests again
2. Checks if previous failures are now passing
3. Updates report with new results
```

### Iteration 2: Refinement (if needed)

**If new issues found or some still failing:**

**Updated report:**
```markdown
# Validation Report - User Profile (Iteration 2)

## Summary
- Total Tests: 15
- Passed: 14
- Failed: 1
- Coverage: 82%

## Iteration Notes
- Previous iteration: 3 failures
- Fixed: 2 (email validation, file size check)
- Still failing: 1 (concurrent updates)
- New failures: 0

## Failed Tests

### Test: test_concurrent_updates
**File:** tests/test_profile.py:89
**Status:** STILL FAILING
**Error:**
```
DatabaseError: Lost update - version mismatch
```
**Issue:** Optimistic locking implemented but version field not incremented
**Previous fix attempt:** Added version field but forgot to increment it
**Fix:** In api/profile.py:updateProfile(), add `version = version + 1` before save

## Progress
✅ Email validation - FIXED
✅ File size check - FIXED
❌ Concurrent updates - Still needs work (version increment missing)
```

Builder fixes the remaining issue, validator re-tests.

### Iteration 3: Final Check (if still needed)

**If STILL failing after iteration 2:**

**Updated report:**
```markdown
# Validation Report - User Profile (Iteration 3 - FINAL)

## Summary
- Total Tests: 15
- Passed: 15 ✅
- Failed: 0
- Coverage: 85%

## Iteration History
- Iteration 1: 3 failures
- Iteration 2: 1 failure
- Iteration 3: 0 failures (ALL PASS)

## Result: VALIDATION PASSED ✅

All acceptance criteria met. Ready for deployment.
```

OR if still failing:

```markdown
# Validation Report - User Profile (Iteration 3 - ESCALATION NEEDED)

## Summary
- Total Tests: 15
- Passed: 14
- Failed: 1
- Coverage: 82%

## Iteration History
- Iteration 1: 3 failures
- Iteration 2: 1 failure
- Iteration 3: 1 failure (NO PROGRESS)

## Result: ESCALATION REQUIRED ❌

After 3 iterations, test_concurrent_updates still failing.

This suggests a fundamental issue that needs human review:
- May need architecture change (optimistic → pessimistic locking)
- May need database upgrade (transactions not supported)
- May need different approach entirely

Recommended actions:
1. Consult with user about acceptable approach
2. Consider alternative solution (e.g., queue-based updates)
3. Accept known issue and document as limitation
```

## Report Structure (Standard Format)

Every validation report must include:

```markdown
# Validation Report - [Feature Name]

## Summary
- Total Tests: X
- Passed: Y
- Failed: Z
- Coverage: XX%

## [Optional] Iteration Notes
(Only in iteration 2+)
- Previous iteration: X failures
- Fixed: Y
- Still failing: Z
- New failures: N

## Test Results

### Passed Tests (summary)
- test_name_1
- test_name_2
...

### Failed Tests (detailed)
For each failure:
- Test name and location
- Error message
- Root cause analysis
- Specific fix recommendation
- Code location to change

## [Optional] Security Issues
Any vulnerabilities found

## [Optional] Coverage Analysis
Files/functions below threshold

## Recommendations
Prioritized list of what to fix

## Result
- PASSED → Ready for deployment
- FAILED → Needs fixes (iteration X of 3)
- ESCALATION → Needs human review
```

## Communication Protocol

### Validator → Builder Messages

**Via report file:**
- Exact test failures with error messages
- Root cause analysis (what's actually wrong)
- Specific fix location (file:line)
- Suggested implementation approach

**What validator should NOT do:**
- Vague descriptions ("tests are failing")
- No context ("fix the code")
- Multiple possible interpretations
- Blame or judgment

### Builder → Validator Confirmation

**Via code changes:**
- Implement fixes in the exact locations mentioned
- Address the specific issues called out
- Add comments if non-obvious

**Via report back:**
- "Fixed 3 issues: [list specifics]"
- "Modified files: [list files]"
- "Ready for re-testing"

## Orchestration Logic

The main conversation (or `/rapid-dev` command) orchestrates:

```python
# Pseudocode for the loop

max_iterations = 3
iteration = 0

while iteration < max_iterations:
    iteration += 1

    # Run validator
    validation_result = invoke_validator_agent(
        feature="user-profile",
        spec_path="docs/specs/user-profile.md",
        iteration=iteration
    )

    # Check result
    if validation_result.all_tests_passed:
        print("✅ Validation passed!")
        break

    if iteration >= max_iterations:
        print(f"❌ After {max_iterations} iterations, tests still failing")
        print("Escalating to user...")
        show_user_options()
        break

    # If failed, invoke builder to fix
    print(f"⚠️ Validation failed ({validation_result.failures_count} issues)")

    builder_result = invoke_builder_agent(
        task=f"Read validation report at {validation_result.report_path} and fix all issues",
        iteration=iteration
    )

    print(f"Builder fixed issues, re-testing (iteration {iteration + 1})...")
    # Loop continues

# After loop
if all_tests_passed:
    proceed_to_deployment()
else:
    ask_user_for_guidance()
```

## File Locations (Convention)

```
docs/
  specs/
    [feature-name].md                    # Created by planner
  validation/
    [feature-name]-report.md             # Created by validator
    [feature-name]-report-iter2.md       # Optional: keep history
    [feature-name]-report-iter3.md       # Optional: keep history
  deployment/
    [feature-name]-staging-[time].md     # Created by shipper
    [feature-name]-production-[time].md  # Created by shipper
  feedback/
    [feature-name]-insights.md           # Created by shipper
```

## Edge Cases

### What if builder can't understand the report?

**Builder should:**
1. Read the report multiple times
2. Search for related code using Grep
3. Read the spec to understand intended behavior
4. Make best effort fix
5. If truly stuck, report back: "Unable to fix issue X - need clarification"

**Main conversation should:**
- Show user the issue
- Get clarification
- Re-invoke builder with additional context

### What if validator generates flaky tests?

**Builder should:**
- Note in response: "Test X appears flaky (timing dependent)"
- Fix what's fixable
- Mark flaky tests for user review

**Validator should (next iteration):**
- Run tests multiple times for flaky ones
- Only fail if consistently failing

### What if there's a regression?

**Validator should:**
- Report new failures clearly: "NEW FAILURE (worked in iteration 1)"
- Help builder understand what broke

**Builder should:**
- Be cautious with changes
- Run local tests before reporting complete

### What if the loop makes no progress?

**After 3 iterations with same failures:**
- STOP automatically
- Show user the stuck issue
- Offer options:
  1. Manual intervention
  2. Simplify the spec
  3. Accept known limitation

## Success Metrics

A good feedback loop should:

✅ **Most features pass in 1-2 iterations** (not always 3)
✅ **Each iteration fixes issues** (measurable progress)
✅ **Reports are actionable** (builder can understand and fix)
✅ **Rare escalations** (< 10% need human intervention)
✅ **Fast cycles** (< 30 min per iteration)

## Benefits

This loop mechanism provides:

1. **Automated quality assurance** - Catches issues without manual testing
2. **Clear communication** - Structured reports are unambiguous
3. **Iterative improvement** - Multiple chances to get it right
4. **Bounded complexity** - Max 3 iterations prevents runaway
5. **Traceable decisions** - All reports saved for posterity
6. **Team collaboration** - Reports readable by humans too

## Integration with Skills

During the loop, agents can use skills:

**Validator can use:**
- `test-coverage-analyzer` - To check coverage depth
- `feedback-analyzer` - To analyze test output patterns

**Builder can use:**
- `tdd-code-generator` - If test-first approach needed
- `modular-code-formatter` - To clean up code after fixes
- `api-client-generator` - If issue is in API client code

## Example: Full Loop Execution

```
User: "Build user profile feature"

/rapid-dev invoked:

1. Planner → Creates docs/specs/user-profile.md
2. Builder → Implements feature
3. Validator → Runs tests → 3 failures → docs/validation/user-profile-report.md

   Iteration 1:
   - Builder reads report
   - Builder fixes 3 issues
   - Validator re-tests → 1 failure remaining

   Iteration 2:
   - Builder reads updated report
   - Builder fixes remaining issue
   - Validator re-tests → ALL PASS ✅

4. Shipper → Deploys to staging → production

Total time: 2 hours (spec + build + 2 validation iterations + deploy)
Result: High-quality feature shipped fast
```

## Parallel Builder-Validator Loops (v2.0)

### How Parallel Loops Work

When building features with multiple modules in parallel, each module gets its own builder-validator feedback loop.

**Key difference:** Loops run **simultaneously** for independent modules.

### Parallel Loop Execution

```
Phase 1 (3 modules in parallel):

Module A:
  Builder A builds → Validator A tests → PASS (1 iteration) ✅
  Duration: 3m 30s

Module B:
  Builder B builds → Validator B tests → PASS (1 iteration) ✅
  Duration: 3m 15s

Module C:
  Builder C builds → Validator C tests → 2 FAILURES
  Iteration 1: Builder C fixes → Validator C re-tests → 1 FAILURE
  Iteration 2: Builder C fixes → Validator C re-tests → PASS ✅
  Duration: 9m 15s (slowest)

All Phase 1 modules validated at: 9m 15s
(vs 16m if sequential)
```

### Multi-Tier Validation Reports

**Tier 1: Module Reports** (per module)
```
docs/validation/
  auth-password-module-report.md       # Module A
  auth-jwt-module-report.md            # Module B
  email-service-module-report.md       # Module C
```

**Tier 2: Integration Report** (after all modules pass)
```
docs/validation/
  auth-integration-report.md           # Tests A+B+C together
```

**Tier 3: System Report** (final comprehensive)
```
docs/validation/
  auth-system-report.md                # Full E2E validation
```

### Module Report Format

```markdown
# Validation Report - [Feature/Module Name] (MODULE-LEVEL)

## Validation Scope
**Level:** Module
**Module:** Module A (Password Validation)
**Phase:** 1
**Validation Mode:** Isolated (dependencies mocked)

## Summary
- Total Tests: 25
- Passed: 25
- Failed: 0
- Coverage: 94%
- Iteration: 1

## Module Status
✅ Module validated successfully

**Next:** Module will proceed to integration phase when all Phase 1 modules pass.
```

### Integration Report Format

```markdown
# Validation Report - [Feature] (INTEGRATION-LEVEL)

## Validation Scope
**Level:** Integration
**Modules Tested:** Module A, Module B, Module C (wired together)
**Validation Mode:** Integration (real dependencies)

## Summary
- Total Integration Tests: 15
- Passed: 15
- Failed: 0

## Integration Points Tested
✅ Module A ↔ Module D: Password validation in auth API
✅ Module B ↔ Module D: Token generation in auth API
✅ Module A ↔ Module E: Password hashing in reset flow
✅ Module C ↔ Module E: Email sending in reset flow

## Integration Status
✅ All modules integrate successfully

**Next:** Proceed to system-level validation
```

### Parallel Loop Orchestration

```python
# Pseudocode for parallel loops

def run_parallel_build(build_plan):
    """Execute parallel build with per-module validation loops."""

    for phase in build_plan.phases:
        print(f"Starting Phase {phase.number}...")

        # Spawn all module builders in parallel
        module_results = {}

        for module in phase.modules:
            # Each module gets its own loop
            result = spawn_parallel(
                module_build_validate_loop,
                module=module,
                max_iterations=3
            )
            module_results[module.name] = result

        # Wait for all modules in phase to complete
        wait_all(module_results)

        # Check if all passed
        if any(not r.passed for r in module_results.values()):
            # Some modules failed after 3 iterations
            failed = [m for m, r in module_results.items() if not r.passed]
            print(f"❌ Phase {phase.number} incomplete:")
            print(f"   Failed modules: {failed}")
            return escalate_to_user(failed_modules=failed)

        print(f"✅ Phase {phase.number} complete (all modules validated)")

    # All phases complete - run integration
    integration_result = run_integration_loop(
        modules=all_modules,
        max_iterations=3
    )

    if not integration_result.passed:
        return escalate_to_user(integration_failed=True)

    print("✅ All modules and integration validated")
    return success()


def module_build_validate_loop(module, max_iterations=3):
    """Run build-validate loop for a single module."""

    iteration = 0
    while iteration < max_iterations:
        iteration += 1

        if iteration == 1:
            # Initial build
            builder_result = build_module(module)
        else:
            # Fix issues from validation
            builder_result = fix_module_issues(
                module=module,
                report=f"docs/validation/{module.name}-report.md"
            )

        # Validate immediately (don't wait for other modules)
        validation_result = validate_module(
            module=module,
            validation_scope="MODULE",  # Isolated testing
            iteration=iteration
        )

        if validation_result.all_passed:
            print(f"✅ {module.name} validated (iteration {iteration})")
            return success(module=module, iterations=iteration)

        if iteration >= max_iterations:
            print(f"❌ {module.name} failed after {max_iterations} iterations")
            return failure(module=module, iterations=iteration)

        print(f"⚠️  {module.name} has issues, fixing (iteration {iteration})...")

    return failure(module=module)
```

### Parallel Loop Benefits

**Speed:**
- Modules validate immediately (no waiting)
- Fast modules complete early
- Slow modules don't block fast ones
- 2-3x faster overall

**Quality:**
- Same 3-iteration limit per module
- Same validation rigor
- Isolation catches module-specific issues
- Integration catches cross-module issues

**Visibility:**
- Per-module progress tracking
- Real-time status dashboard
- Clear attribution of issues
- Parallel validation reports

### Handling Parallel Loop Failures

**Scenario:** Module A and B pass, Module C fails after 3 iterations

**Options:**

1. **Continue with successful modules:**
   - Proceed to Phase 2 with A and B
   - Build dependent modules using A+B
   - Fix Module C separately, add later

2. **Block and escalate:**
   - Don't proceed to Phase 2
   - User fixes Module C manually
   - Re-run validation
   - Continue when C passes

3. **Simplify feature:**
   - Remove Module C from feature
   - Adjust dependent modules
   - Ship without C, add in iteration 2

**Default:** Block and escalate (option 2) - maintain quality gate.

### Integration Loop After Parallel Modules

After all modules in all phases pass their individual loops:

```python
def run_integration_loop(modules, max_iterations=3):
    """Run integration validation loop."""

    iteration = 0
    while iteration < max_iterations:
        iteration += 1

        if iteration == 1:
            # Initial integration
            integration_builder = wire_modules_together(modules)
        else:
            # Fix integration issues
            integration_builder = fix_integration_issues(
                report="docs/validation/[feature]-integration-report.md"
            )

        # Run integration-level validation
        validation = validate_integration(
            modules=modules,
            validation_scope="INTEGRATION",
            iteration=iteration
        )

        if validation.all_passed:
            return success()

        if iteration >= max_iterations:
            return escalate_to_user()

    return failure()
```

### Comparison: Sequential vs Parallel Loops

**Sequential (v1.x):**
```
Builder builds everything → Validator tests everything
If failures: Builder fixes → Validator re-tests
Max 3 iterations for entire feature
Total: ~25 minutes
```

**Parallel (v2.0):**
```
Phase 1 (parallel):
  Builder A → Validator A (3m)
  Builder B → Validator B (3m)
  Builder C → Validator C (9m with 2 iterations)

Phase 2 (parallel):
  Builder D → Validator D (4m)
  Builder E → Validator E (4m)

Integration:
  Integration Builder → Integration Validator (2m)

Total: ~12 minutes (2.1x faster)
```

**Key:** Each module's loop is independent, failures in one don't block others.

---

## Conclusion

The feedback loop is the secret sauce that enables "rapid agile at vibe speeds":

- **Fast**: Automated, no waiting for humans
- **Reliable**: Catches issues before production
- **Scalable**: Works for any feature size (now even faster with parallel)
- **Traceable**: All communication documented
- **🆕 Parallel**: Multiple loops run simultaneously for complex features

By using files as the communication mechanism, we get the benefits of async, persistent, structured communication between agents - now with parallel execution for maximum speed.
