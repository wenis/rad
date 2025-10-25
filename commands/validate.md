---
name: validate
description: Invoke the validator agent to test and verify code quality
---

# Validate Code

Invoke the validator agent to thoroughly test code and check quality.

## Usage

Use this command when you want to:
- Run comprehensive tests on new code
- Check test coverage
- Verify acceptance criteria are met
- Find security issues or bugs
- Get validation report before deployment

## What This Does

The validator agent will:
1. Analyze the code
2. Generate tests if missing
3. Run all tests (unit, integration, edge cases)
4. Check for security issues
5. Verify against spec (if available)
6. Create detailed validation report at `docs/validation/[feature]-report.md`

## Instructions

Invoke the **validator** agent with:
- Feature/component to validate
- Path to spec (if available)
- Any specific concerns (security, performance, etc.)

The validator will report back with:
- Test results (passed/failed)
- Issues found (critical, warning, suggestion)
- Coverage metrics
- Path to full validation report
- Recommendation (fix issues or proceed to deployment)

## Example Flow

**User says:** "Validate the user profile feature"

**You do:**
1. Invoke validator agent
2. Point it to the code and spec
3. Validator runs tests
4. Validator creates report

**If tests pass:**
- Suggest running `/ship` to deploy
- Or mark the feature as complete

**If tests fail:**
- Validator creates detailed failure report
- Run `/build` again, telling builder to read the validation report
- Builder fixes issues
- Run `/validate` again
- Repeat until tests pass (max 3 iterations)

## Validation Loop (Important!)

When validation finds issues:

```
Iteration 1:
/validate → Finds 3 failing tests → Creates report

/build "Read validation report at docs/validation/user-profile-report.md and fix the issues"
→ Builder reads report
→ Builder fixes specific issues
→ Builder reports fixes complete

/validate → Re-run tests
→ All pass or new issues found

Iteration 2 (if needed):
Repeat above process

Iteration 3 (if still failing):
Ask user for guidance - something fundamental may be wrong
```

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

## Next Steps

**If validation passes:**
- Run `/ship` to deploy
- Or mark feature complete

**If validation fails:**
- Run `/build` with the validation report
- Fix issues
- Run `/validate` again
- Repeat max 3 times

**If still failing after 3 iterations:**
- Consult with user
- May need to revisit spec or approach
