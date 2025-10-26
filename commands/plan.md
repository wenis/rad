---
name: plan
description: Invoke the planner agent to understand project state and plan features
---

# Plan Feature

Invoke the planner agent to understand your project and plan what to work on next.

## What This Does

The planner agent is **context-aware**. It will:

**If you provide a feature description:**
- Create structured specs with user stories and acceptance criteria
- Define test scenarios and dependencies
- Write spec to `docs/specs/[feature-name].md`

**If you DON'T provide a feature description:**
- Read `docs/SYSTEM.md` to understand your project
- Review existing specs in `docs/specs/` to see what's been planned
- Review validation reports to see what's been built
- **Recommend what to work on next** based on project state
- Suggest next steps (new features, incomplete work, etc.)

## Instructions

**ALWAYS invoke the planner agent immediately.** Do not wait for more input.

**Pass to the planner:**
1. The user's feature description (if provided)
2. Context that you were invoked via `/plan` command
3. All relevant conversation context

**The planner will:**
- Read system context (`docs/SYSTEM.md`, `docs/CONVENTIONS.md`)
- Understand project state (specs, validation reports)
- Either create a new spec OR recommend what to work on next
- Report back with clear next steps

## Example: With Feature Description

**User says:** `/plan add user profile page`

**Planner does:**
1. Reads `docs/SYSTEM.md` to understand tech stack
2. Creates `docs/specs/user-profile.md` with requirements
3. Recommends: "Run `/build` to implement"

## Example: Without Feature Description (Resuming Work)

**User says:** `/plan`

**Planner does:**
1. Reads `docs/SYSTEM.md` to understand what we're building
2. Checks `docs/specs/` to see what's been planned
3. Checks `docs/validation/` to see what's been built
4. Reports: "I see we're building TaskFlow. You have 3 features planned:
   - ✅ User auth (completed)
   - ⏸️ Task management (spec exists, not built yet)
   - ⏸️ Real-time collaboration (spec exists, not built yet)

   Recommended: Run `/build` to implement task management"

## Next Steps

After planning:
- Review the planner's recommendations
- If creating a new feature, run `/build` to implement
- If resuming work, follow the planner's suggested next steps
