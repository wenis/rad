---
name: build
description: Invoke the builder agent to implement features from specs
---

# Build Feature

Invoke the builder agent to implement a feature from specifications.

## Usage

Use this command when you want to:
- Implement a feature from a spec
- Prototype a new idea quickly
- Refactor existing code
- Fix a bug

## What This Does

The builder agent will:
1. Read the spec (if available)
2. Implement the feature
3. Write clean, modular code
4. Run basic smoke tests
5. Report back with what was built

## Instructions

**CRITICAL:** When invoking the builder agent, you MUST use a minimal prompt that allows the builder to follow its own mode detection process. DO NOT give implementation instructions.

**Correct approach - Invoke builder agent with minimal prompt:**

```
Read the spec at {spec_path} and implement the feature.

The builder agent will handle mode detection (sequential vs parallel orchestration) autonomously.
```

**WRONG approach - DO NOT do this:**
```
❌ Implement the feature by doing X, Y, Z... (too prescriptive)
❌ Begin implementation now... (bypasses mode detection)
❌ Create files A, B, C... (skips orchestration)
```

The builder will implement the feature and report back with:
- Files created/modified
- How to test the implementation
- Any issues encountered
- Recommendation to run validation

## With Spec (Recommended)

If a spec exists (e.g., from `/plan`):
```
Read the spec at docs/specs/user-profile.md and implement the feature.
```

**Let the builder agent handle everything else** - it will:
1. Detect if spec requires parallel or sequential execution
2. Enter appropriate mode (Orchestration, Direct Build, Module, or Integration)
3. Spawn sub-agents if needed for parallel execution
4. Coordinate validation loops
5. Report results

## Without Spec (Quick Prototype)

For quick prototyping without formal spec:
```
Implement a user profile page with name, email, and avatar upload.
```

## Example Flow

**User says:** "Now build the user profile feature we planned"

**You do:**
1. Invoke builder agent with minimal prompt:
   ```
   Read the spec at docs/specs/user-profile.md and implement the feature.
   ```
2. Builder detects execution strategy (parallel or sequential)
3. Builder implements feature (may spawn sub-agents if parallel)
4. Builder reports what was created
5. Suggest running `/validate` next

**DO NOT:**
- Add detailed implementation steps to the builder prompt
- Tell builder what files to create
- Tell builder what functions to implement
- Say "Begin implementation now"

## Reading Validation Reports

If this is a fix iteration (validator found issues):
- Tell builder to read the validation report at `docs/validation/[feature]-report.md`
- Builder will see what failed and why
- Builder will fix the specific issues identified

## Next Steps

After building:
- Review code with user if needed
- Run `/validate` to test thoroughly
- If validation fails, run `/build` again with the validation report
- Or continue with `/rapid-dev` for full cycle
