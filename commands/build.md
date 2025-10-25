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

Invoke the **builder** agent with:
- Path to spec file (if one exists)
- Feature description
- Any constraints or preferences

The builder will implement the feature and report back with:
- Files created/modified
- How to test the implementation
- Any issues encountered
- Recommendation to run validation

## With Spec (Recommended)

If a spec exists (e.g., from `/plan`):
```
Builder: Read spec at docs/specs/user-profile.md and implement the feature
```

## Without Spec (Quick Prototype)

For quick prototyping without formal spec:
```
Builder: Implement a user profile page with name, email, and avatar upload
```

## Example Flow

**User says:** "Now build the user profile feature we planned"

**You do:**
1. Invoke builder agent
2. Point it to `docs/specs/user-profile.md`
3. Builder implements the feature
4. Builder reports what was created
5. Suggest running `/validate` next

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
