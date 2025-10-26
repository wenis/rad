# Slash Command Best Practices

This document explains how to write effective slash commands that properly delegate to agents without bypassing their autonomy.

---

## 🚨 CRITICAL WARNING: Minimal Prompts Required

**If you're experiencing issues with parallel builds not working, READ THIS:**

The RAD system uses **command-level routing** to determine which agent to invoke:
- **Parallel strategy** → Command invokes **orchestrator agent** (coordinates multiple builders)
- **Sequential strategy** → Command invokes **builder agent** (builds directly)

**The command reads the spec to detect the strategy, then routes to the appropriate agent.**

**✅ CORRECT approach:**
```
1. Command reads spec file
2. Command finds "Execution Strategy: Parallel" in spec
3. Command invokes orchestrator with: "Orchestrate the build for docs/specs/feature.md"
4. Orchestrator spawns multiple builders, coordinates validation, handles integration
```

**❌ WRONG approach:**
```
1. Command ignores spec strategy
2. Command invokes builder with detailed instructions
3. Builder tries to do everything itself
4. Result: Sequential build when parallel was intended
```

**Key principles:**
- **Commands are smart:** They read specs and route to appropriate agents
- **Agents are focused:** Builder builds, orchestrator orchestrates
- **Prompts are minimal:** < 20 words, no detailed instructions

**If parallel builds aren't working:**
1. Check that command is reading the spec
2. Verify command is routing to orchestrator (not builder) for parallel specs
3. Ensure agent prompts are minimal

---

## Core Principle: Minimal Prompts

**Golden Rule:** Commands should invoke agents with **minimal, non-prescriptive prompts** that allow agents to use their own decision-making logic.

### Why This Matters

The RAD system separates concerns:
- **Commands:** Read specs, route to appropriate agents
- **Orchestrator:** Coordinates parallel builds (spawns builders, validators, integration)
- **Builder:** Implements features (writes code, fixes issues)
- **Validator:** Tests features (runs tests, reports issues)
- **Planner:** Creates specs (designs features, defines build strategies)
- **Shipper:** Deploys features (staging, production, monitoring)

When commands generate overly prescriptive prompts, they **bypass** this architecture, leading to:
- ❌ Wrong agent invoked (builder instead of orchestrator)
- ❌ Agents receiving detailed instructions instead of spec paths
- ❌ Parallel builds becoming sequential
- ❌ Loss of agent autonomy and intelligence
- ❌ Inconsistent behavior across commands
- ❌ Harder to debug when things go wrong

## Command Design Patterns

### ✅ GOOD: Minimal Delegation

**Pattern:**
```markdown
## Instructions

Invoke the **[agent-name]** agent with:
- [Required context: spec path, feature name, etc.]
- [Any user-specified constraints]

**Example invocation:**
```
[Agent]: [Minimal instruction like "Read spec at path/to/spec.md and implement the feature"]
```
```

**Why this works:**
- Agent receives necessary context
- Agent makes its own decisions about HOW to proceed
- Agent follows its documented behavior patterns
- User gets consistent agent behavior

**Example from `/build`:**
```markdown
## With Spec (Recommended)

If a spec exists (e.g., from `/plan`):
```
Read the spec at docs/specs/user-profile.md and implement the feature.
```

Let the builder agent handle everything else - it will:
1. Detect if spec requires parallel or sequential execution
2. Enter appropriate mode (Orchestration, Direct Build, Module, or Integration)
3. Spawn sub-agents if needed for parallel execution
4. Coordinate validation loops
5. Report results
```

### ❌ BAD: Overly Prescriptive

**Anti-pattern:**
```markdown
**IF spec says "Parallel":**
Invoke builder agent with:
"Build feature from spec.

Mode: Orchestration (REQUIRED)

You MUST orchestrate parallel builders according to the build plan.
The spec specifies a parallel build strategy - you must spawn multiple
builder agents for each phase as defined in the build plan.

DO NOT build everything yourself sequentially.

Begin implementation now."
```

**Why this fails:**
- ❌ Tells agent what mode to use (bypasses mode detection)
- ❌ Explains how to orchestrate (agent already knows)
- ❌ Adds unnecessary directives ("DO NOT...")
- ❌ "Begin implementation now" skips critical thinking steps
- ❌ Creates inconsistency (different from direct agent invocation)

### 🟡 ACCEPTABLE: Conditional Context

Sometimes you need to provide conditional context based on user input:

```markdown
**If this is a fix iteration:**
Invoke builder agent with:
"Read the validation report at docs/validation/[feature]-report.md and fix the issues."

**If this is a new feature:**
Invoke builder agent with:
"Read the spec at docs/specs/[feature].md and implement the feature."
```

**Why this is OK:**
- ✅ Still minimal (just pointing to different input files)
- ✅ Doesn't tell agent HOW to fix or build
- ✅ Lets agent make decisions about approach
- ✅ Context is relevant and necessary

## Agent Invocation Checklist

When writing a command that invokes an agent, ask:

1. **Is the prompt minimal?**
   - Does it just point the agent to inputs (specs, reports, code)?
   - Or does it explain HOW to do the work?

2. **Does it bypass agent logic?**
   - Does it tell the agent what mode to use?
   - Does it explain the agent's own procedures?
   - Does it add directives the agent already knows?

3. **Is it consistent?**
   - Would invoking the agent directly give the same behavior?
   - Or does the command create special behavior?

4. **Does it include unnecessary imperatives?**
   - "You MUST..."
   - "DO NOT..."
   - "Begin implementation now..."
   - These usually indicate over-prescription

5. **Could this be simpler?**
   - Can you remove words without losing meaning?
   - Are you explaining things the agent already knows?

## Common Mistakes

### Mistake 1: Duplicating Agent Instructions

**Bad:**
```
Invoke planner agent with:
"Plan the feature. Create user stories, acceptance criteria, test scenarios,
and a build plan with execution strategy. Write to docs/specs/[feature].md"
```

**Good:**
```
Invoke planner agent with:
"Plan the [feature-name] feature."
```

The planner already knows to create user stories, acceptance criteria, etc. That's documented in `agents/planner.md`.

### Mistake 2: Telling Agents Their Mode

**Bad:**
```
Invoke builder in Orchestration Mode:
"You are in orchestration mode. Spawn parallel builders for each module..."
```

**Good:**
```
Invoke builder agent with:
"Read the spec at docs/specs/[feature].md and implement the feature."
```

The builder detects its own mode by reading the spec's execution strategy.

### Mistake 3: Adding Conditional Logic That Belongs in the Agent

**Bad:**
```
IF spec has parallel strategy:
  Tell builder: "Use parallel orchestration with X modules..."
ELSE:
  Tell builder: "Build sequentially..."
```

**Good:**
```
Tell builder: "Read the spec and implement the feature."
```

The builder reads the spec itself and decides orchestration vs sequential.

### Mistake 4: Directives That Override Agent Judgment

**Bad:**
```
"Implement the feature. Use FastAPI for the backend. Create REST endpoints.
Add JWT authentication. Deploy to AWS. Use PostgreSQL for the database."
```

**Good:**
```
"Implement the feature from the spec."
```

The spec already contains tech stack info. The builder reads SYSTEM.md for tech decisions.

## Command Structure Template

```markdown
---
name: command-name
description: Brief description of what this command does
---

# Command Title

Brief overview of purpose.

## When to Use

- Situation 1
- Situation 2
- Situation 3

## What This Does

The [agent-name] agent will:
1. [What it does - NOT how]
2. [What it produces]
3. [What it reports back]

## Instructions

Invoke the **[agent-name]** agent with:
- [Minimal context needed]
- [User preferences if any]

**Example invocation:**
```
[Minimal instruction to agent]
```

The agent will handle:
- [Thing 1 the agent decides]
- [Thing 2 the agent decides]
- [Thing 3 the agent decides]

## Example Flow

**User says:** "[User request]"

**You do:**
1. Invoke [agent] with: "[Minimal prompt]"
2. Agent performs work autonomously
3. Agent reports results
4. Suggest next step

## Next Steps

After [agent] completes:
- [What to do next]
- [How to proceed]
```

## Testing Your Command

Before committing a new command:

1. **Manual test:** Use the command and verify agent behaves as expected
2. **Mode detection test:** For builder commands, verify parallel specs trigger orchestration
3. **Consistency test:** Compare command invocation to direct agent invocation - same behavior?
4. **Minimal prompt test:** Remove words from the prompt. Does it still work? Keep removing until it breaks.

## Examples

### ✅ Good Command: /validate

```markdown
## Instructions

Invoke the **validator** agent with:
- Feature/component to validate
- Path to spec (if available)
- Any specific concerns (security, performance, etc.)
```

**Why it's good:**
- Minimal context (what to validate, where spec is)
- Doesn't tell validator HOW to validate
- Lets validator decide what tests to run
- Validator's own logic determines test suite

### ✅ Good Command: /plan

```markdown
## Instructions

**ALWAYS invoke the planner agent immediately.** Do not wait for more input.

**Pass to the planner:**
1. The user's feature description (if provided)
2. Context that you were invoked via `/plan` command
3. All relevant conversation context
```

**Why it's good:**
- Directive to invoke immediately (prevents delay)
- Passes minimal context (feature description)
- Lets planner decide how to plan
- Planner reads SYSTEM.md and makes tech decisions

### ✅ Good Command: /build (after fix)

```markdown
## Instructions

**CRITICAL:** When invoking the builder agent, you MUST use a minimal prompt that allows the builder to follow its own mode detection process. DO NOT give implementation instructions.

**Correct approach:**
```
Read the spec at {spec_path} and implement the feature.
```

**WRONG approach - DO NOT do this:**
```
❌ Implement the feature by doing X, Y, Z... (too prescriptive)
❌ Begin implementation now... (bypasses mode detection)
❌ Create files A, B, C... (skips orchestration)
```
```

**Why it's good:**
- Explicitly warns against over-prescription
- Shows correct minimal pattern
- Shows anti-patterns to avoid
- Lets builder handle mode detection

## Updating Existing Commands

If you find a command that's overly prescriptive:

1. **Identify the agent being invoked**
2. **Read the agent's prompt** (`agents/[name].md`)
3. **Understand agent's built-in logic** (what it already knows)
4. **Remove duplicate instructions** (don't repeat what agent knows)
5. **Simplify to minimal context** (just point to inputs)
6. **Test the simplified version** (verify agent behavior)

## Related Documentation

- `agents/README.md` - How agents work and their capabilities
- `PHILOSOPHY.md` - When agents should be strict vs flexible
- `LOOP-MECHANISM.md` - How builder-validator feedback loop works
- `docs/templates/` - Templates for specs, ADRs, etc.

## Questions?

If unsure whether a command is too prescriptive, ask:

> "If I removed this instruction, would the agent still know what to do?"

If yes → remove it. The agent has that logic built-in.

---

**Last Updated:** 2025-10-25
**Maintainer:** RAD System Contributors
