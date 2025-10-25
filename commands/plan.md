---
name: plan
description: Invoke the planner agent to create specifications from requirements
---

# Plan Feature

Invoke the planner agent to translate ideas and requirements into structured specifications.

## Usage

Use this command when you want to:
- Create specs for a new feature
- Break down a large feature into tasks
- Clarify vague requirements
- Document acceptance criteria

## What This Does

The planner agent will:
1. Gather requirements from your description
2. Create user stories and acceptance criteria
3. Define test scenarios
4. Identify risks and dependencies
5. Write a spec document to `docs/specs/[feature-name].md`

## Instructions

Invoke the **planner** agent with the user's feature request or requirements.

Pass all context from the user about:
- Feature description
- Target users
- Business goals
- Constraints or requirements
- Any existing specs or related features

The planner will create a comprehensive spec document and report back with:
- Path to the created spec file
- Summary of user stories
- Key acceptance criteria
- Recommended next steps (usually: invoke builder)

## Example

**User says:** "I want to add a user profile page where users can update their name, email, and avatar"

**You do:**
1. Invoke planner agent with this requirement
2. Planner creates `docs/specs/user-profile.md`
3. Review spec with user
4. If approved, suggest running `/build` next

## Next Steps

After planning:
- Review the spec with the user
- Make adjustments if needed
- When approved, run `/build` to implement
- Or run `/rapid-dev` to do the full cycle
