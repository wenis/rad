---
name: validate
description: Manages the builder-validator feedback loop with automatic iteration tracking. Tests code quality, coordinates fixes, and ensures max 3 iterations before user escalation.
---

# Validate Code

Manage the builder-validator feedback loop to test code quality and coordinate fixes automatically.

**Why this command exists:** Quality gates prevent bugs from reaching production, but manual back-and-forth between validation and fixing is slow and error-prone. This command automates the feedback loop: validator finds issues → user chooses action → builder fixes → validator retests → repeat max 3 times. This ensures consistent quality while maintaining rapid iteration speed.

## Usage

Use this command when you want to:
- Run comprehensive tests on new code (**Why:** Catch bugs before they reach production)
- Check test coverage (**Why:** Untested code is likely to have hidden bugs)
- Verify acceptance criteria are met (**Why:** Ensures feature matches spec requirements)
- Find security issues or bugs (**Why:** Security vulnerabilities must be caught before deployment)
- Get validation report before deployment (**Why:** Quality gate ensures production readiness)

## What This Does

The validator agent will:
1. Analyze the code
2. Generate tests if missing
3. Run all tests (unit, integration, edge cases)
4. Check for security issues
5. Verify against spec (if available)
6. Create detailed validation report at `docs/validation/[feature]-report.md`

## Instructions

**Follow these steps to handle validation and feedback loop automatically:**

### Step 1: Invoke Validator

**Why this step:** Validator agent has the tools and expertise to generate comprehensive tests, run them, and create detailed reports. This step initiates the quality check process.

Invoke the **validator** agent with:
- Feature/component to validate
- Path to spec (if available)
- Any specific concerns (security, performance, etc.)

The validator will report back with:
- Test results (passed/failed)
- Issues found (critical, warning, suggestion)
- Coverage metrics
- Path to full validation report

### Step 2: Check Validation Results

**Why this step:** You need to determine if fixes are required before proceeding. Reading the report yourself (rather than blindly trusting the validator's summary) ensures you understand the issue severity and can make informed decisions about next steps.

After validator completes, read the validation report to determine next action:

**If all tests pass and no critical issues:**
- ✅ Report success to user
- ✅ Suggest next step: `/ship` for deployment OR mark feature complete
- **Why:** No blockers exist; code meets quality gates

**If critical issues or failing tests found:**
- ❌ Note the issues
- ❌ Track iteration number (1, 2, or 3)
- ❌ Proceed to Step 3 (Feedback Loop)
- **Why:** Critical issues block deployment; must be fixed before proceeding

### Step 3: Automatic Feedback Loop (Max 3 Iterations)

**Why this step:** Automated loops reduce friction and ensure consistent process. Max 3 iterations prevents infinite loops when fundamental architectural problems exist.

**When validation fails, automatically handle the feedback loop:**

1. **Ask user if they want to fix issues:**

   **Why ask:** User may want manual control (to learn), may accept current state (known issues acceptable), or may want automated fix (most common). Asking preserves user agency.

   Use AskUserQuestion with options:
   - "Fix issues now (invoke builder)" - Most common choice
   - "Fix manually (I'll edit code myself)" - User wants control
   - "Skip for now (deploy anyway)" - Accept current state

2. **If user chooses "Fix issues now":**

   **Why invoke builder:** Builder agent specializes in code fixes and can read validation reports to make targeted changes. This is faster and more reliable than manual fixes for most issues.

   Invoke builder agent with validation report:
   ```
   Read validation report at docs/validation/[feature]-report.md and fix all issues.
   ```

3. **After builder completes fixes:**

   **Why auto re-validate:** Immediate re-testing provides fast feedback on whether fixes worked. Waiting would slow iteration cycles and risk forgetting context.

   Automatically re-invoke validator (same scope as before)

4. **Track iterations:**

   **Why track:** Prevents infinite loops and signals when architectural changes are needed rather than tactical fixes.

   - Iteration 1: First validation failure → Fix → Re-validate
   - Iteration 2: Still failing → Fix again → Re-validate
   - Iteration 3: Still failing → Report to user for guidance

5. **After iteration 3 with failures:**

   **Why stop at 3:** If 3 fix attempts failed, the problem is likely architectural (wrong approach) or requirements are unclear. Continuing wastes time; better to escalate for human judgment.

   Stop the loop and report:
   ```
   ⚠️ After 3 iterations, validation still failing.

   Remaining issues: [summary]

   This suggests:
   - Architectural problem (may need to revisit spec)
   - Requirements unclear (may need user clarification)
   - Complexity underestimated (may need more time)

   What would you like to do?
   - Revisit the spec
   - Continue fixing manually
   - Accept current state with known issues
   ```

## Example Flow

**Scenario: Validation passes**

```
User: "Validate the user profile feature"

You: [Invoke validator agent]
Validator: ✅ All 23 tests passing, coverage 87%, no critical issues
You: "✅ Validation passed! Ready for deployment. Run `/ship` when ready."
```

**Scenario: Validation fails (with automatic feedback loop)**

```
User: "Validate the authentication feature"

You: [Invoke validator - Iteration 1]
Validator: ❌ 3 failing tests, 2 critical security issues
You: [Read report, note issues]
You: [Use AskUserQuestion]
   Question: "Validation found 2 critical issues. How should we proceed?"
   Options:
   - Fix issues now (invoke builder)
   - Fix manually
   - Skip for now

User: [Selects "Fix issues now"]

You: [Invoke builder]
   Prompt: "Read validation report at docs/validation/auth-feature-report.md and fix all issues."
Builder: 🔧 Fixed 3 issues (added input validation, fixed password check, added tests)

You: [Auto re-invoke validator - Iteration 2]
Validator: ✅ All 26 tests passing
You: "✅ All issues fixed! Validation passed after 1 iteration. Ready for `/ship`."
```

**Scenario: Multiple iterations needed**

```
User: "Validate the payment processing feature"

Iteration 1:
Validator: ❌ 5 failing tests
You: [Ask user] → User chooses "Fix now"
Builder: Fixed 4 issues
Validator: ❌ 2 tests still failing (1 was missed, 1 is new)

Iteration 2:
You: [Ask user again] → User chooses "Fix now"
Builder: Fixed 2 remaining issues
Validator: ✅ All tests passing
You: "✅ Validation passed after 2 iterations!"
```

**Scenario: Iteration 3 exhausted**

```
Iteration 1: Validator fails → Builder fixes → Still failing
Iteration 2: Builder fixes again → Still failing
Iteration 3: Builder fixes third time → Still failing

You: "⚠️ After 3 iterations, validation still failing.

Remaining issues:
- Database transaction deadlock under load (architectural issue)
- Race condition in payment confirmation (needs redesign)

This suggests fundamental architectural problems. Options:
- Revisit the spec and redesign approach
- Continue fixing manually with user guidance
- Accept current state and document known limitations"
```

**Scenario: User chooses "Fix manually"**

```
User: "Validate the data import feature"

You: [Invoke validator]
Validator: ❌ 2 failing tests - CSV parsing errors
You: [Use AskUserQuestion]
     "Validation found 2 critical issues. How should we proceed?"
     [Fix now] [Fix manually] [Skip]

User: [Selects "Fix manually" - wants to learn]

You: "The validation report is at docs/validation/data-import-report.md

      Issues to fix:
      1. CSV parser fails on empty fields (line 45 in parser.py)
      2. Date format validation too strict (line 78 in parser.py)

      After fixing, run `/validate` again to verify."

[User edits code manually]

User: "/validate data import"

You: [Re-invoke validator]
Validator: ✅ All tests passing
You: "✅ Issues fixed! Validation passed."
```

**Scenario: Partial success (some issues fixed, some remain)**

```
Iteration 1:
Validator: ❌ 5 failing tests
Builder: Fixed 3 issues
Validator: ❌ 2 tests still failing + 1 new issue from fix

You: "Iteration 1 made progress: 3/5 issues fixed, but 2 remain + 1 new issue introduced.

Still failing:
- Memory leak in connection pooling (original issue)
- Invalid error handling (original issue)
- Missing null check (new - introduced by fix)

[Ask user] → User chooses 'Fix now'"

Iteration 2:
Builder: Fixed all 3 remaining issues
Validator: ✅ All tests passing
You: "✅ All issues resolved after 2 iterations!"
```

## Output Format Guidance

**When reporting validation results to user, use consistent format:**

**For successful validation:**
```
✅ Validation Complete: [Feature Name]

Summary:
- Tests: X/X passing (100%)
- Coverage: XX%
- No critical issues found

Status: Ready for deployment
```

**For failed validation (after asking user):**
```
❌ Validation Failed: [Feature Name]

Summary:
- Tests: X/Y passing (Z%)
- Critical issues: N

Iteration X of 3: [Invoking builder to fix issues...]
OR
User chose to fix manually. Report: docs/validation/[feature]-report.md
```

**For iteration completion:**
```
✅ All issues resolved after N iteration(s)!

Final results:
- Tests: X/X passing (100%)
- Coverage: XX%

Status: Ready for `/ship`
```

**Why consistent format:**
- Users can quickly scan status (✅ or ❌)
- Iteration tracking visible at a glance
- Clear next steps provided
- Standardized across all validations

## Key Principles

**Automatic feedback loop:**
- Command handles the builder-validator loop automatically
- User only chooses: fix now, fix manually, or skip
- Max 3 iterations before escalating to user

**Why automatic:**
- Reduces friction (user doesn't need to remember commands)
- Tracks iteration count automatically
- Ensures consistent flow across all validations
- Faster iteration cycles

**Why ask user:**
- User may want to fix manually (learning, specific approach)
- User may want to deploy with known issues (acceptable risk)
- User retains control of the process

## Validation Report Location

The validator always writes a detailed report to:
```
docs/validation/[feature-name]-report.md
```

This report includes:
- Test results summary
- Specific failures with reproduction steps
- Fix recommendations
- Coverage analysis
- Security findings

## Quality Gates

The validator enforces these quality gates:
- ✅ All critical tests must pass
- ✅ No security vulnerabilities
- ✅ Acceptance criteria met (if spec exists)
- ⚠️ Coverage should be > 70% (warning if lower)

## Summary

The `/validate` command provides **automatic feedback loop management**:

1. **Invoke validator** → Creates detailed report
2. **Check results** → Pass or fail?
3. **If fails** → Ask user: fix now, fix manually, or skip
4. **If "fix now"** → Auto-invoke builder with report
5. **After builder** → Auto re-validate
6. **Repeat** → Max 3 iterations
7. **If still failing** → Escalate to user with guidance

**This ensures:**
- ✅ Consistent validation process
- ✅ Automatic iteration tracking
- ✅ User maintains control
- ✅ Clear path forward at each step
