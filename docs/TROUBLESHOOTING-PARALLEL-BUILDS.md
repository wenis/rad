# Troubleshooting Parallel Builds

## Problem: Orchestrator Not Being Invoked for Parallel Builds

If you invoke `/build` on a spec with parallel execution strategy, but parallel builders aren't being spawned, follow this diagnostic guide.

**Important:** The RAD system now uses command-level routing. Commands read the spec and route to either:
- **orchestrator agent** (for parallel builds)
- **builder agent** (for sequential builds)

This troubleshooting guide helps diagnose routing and orchestration issues.

## Quick Diagnosis

### Step 1: Check Which Agent Was Invoked

When you invoke `/build`, Claude Code shows which agent is being invoked. Look for this in the output:

```
orchestrator(Orchestrate the build for...)
  OR
builder(Read the spec at...)
```

**Which agent was invoked?**
- **orchestrator** → Correct for parallel builds, check Step 3
- **builder** → **THIS IS THE PROBLEM** → Should have invoked orchestrator, see Fix below

### Step 2: Check the Routing Logic

The `/build` command should:
1. Read the spec file
2. Find "Execution Strategy" section
3. Route based on strategy:
   - "Parallel" → Invoke orchestrator
   - "Sequential" → Invoke builder

**Did the command read the spec first?**
- ✅ YES → Command is working correctly
- ❌ NO → **THIS IS THE PROBLEM** → Command skipped routing logic, see Fix below

**What was the actual routing decision?**
- Check command output for: "Found: Execution Strategy: Parallel" or similar
- If command didn't output routing decision → It may have skipped reading spec

### Step 3: Check the Spec

Open the spec file and look for the "Build Plan" or "Execution Strategy" section:

**Good (will trigger parallel):**
```markdown
## Build Plan

**Execution Strategy:** Parallel (3 modules in 2 phases)

### Phase 1: Foundation (Parallel)
- Module A: Base collector
- Module B: Parser
- Module C: Validator

### Phase 2: Integration
- Module D: Integration layer
```

**Bad (will be sequential):**
```markdown
## Build Plan

**Execution Strategy:** Sequential (single builder)
```

If the spec doesn't have "Parallel" in execution strategy, the builder will correctly use Direct Build Mode.

## The Fix

### If Problem is the Prompt (Steps 1-2)

Claude Code is adding extra context to the builder prompt. This was fixed in recent versions of RAD.

**Update your RAD system:**
1. Pull latest changes from RAD repo
2. Copy updated command files to your project's `.claude/commands/`
3. Try `/build` again

**Temporary workaround:**
Instead of using `/build`, invoke the builder agent directly:
```
@builder Read the spec at docs/specs/SPEC-XXX.md and implement the feature.
```

### If Problem is the Spec (Step 3)

The spec doesn't define a parallel execution strategy. Either:
1. The feature doesn't need parallel execution (this is correct behavior)
2. The planner didn't detect parallelization opportunity (re-run `/plan` with more detail)

## Expected Behavior

When a parallel build works correctly, you'll see:

```
📋 BUILD PLAN DETECTED

Strategy: Parallel (3 modules in 2 phases)
Mode: Orchestration

Parallel Execution Plan:
- Phase 1: 3 modules
  • Module A: Base collector
  • Module B: Parser
  • Module C: Validator

Spawning 3 parallel builders for Phase 1 now...
```

Then you'll see multiple Task tool invocations in a single message.

## What NOT to See

If the builder starts using Write or Edit tools directly without announcing orchestration mode, **mode detection failed**.

## Root Cause Explanation

The RAD system uses command-level routing:

1. **Command** receives user request
2. **Command** reads the spec file
3. **Command** finds "Execution Strategy" section
4. **If "Parallel"** → Command invokes **orchestrator** agent
5. **If "Sequential"** → Command invokes **builder** agent

**Orchestrator agent** (for parallel builds):
- Parses build plan
- Spawns multiple builder sub-agents via Task tool
- Monitors progress
- Coordinates validation loops
- Handles integration

**Builder agent** (for sequential builds):
- Reads spec
- Implements feature directly
- Uses Write/Edit tools
- Reports completion

**If the wrong agent is invoked:**
- Builder invoked for parallel spec → Builds sequentially (slow, no parallelization)
- Orchestrator invoked for sequential spec → Unnecessary overhead

## How to Prevent This

When writing custom commands:

**✅ Correct approach:**
```markdown
1. Read the spec file
2. Find "Execution Strategy" section
3. If "Parallel" → Invoke orchestrator: "Orchestrate the build for {spec_path}"
4. If "Sequential" → Invoke builder: "Read the spec at {spec_path} and implement the feature."
```

**❌ Wrong approach:**
```markdown
1. Invoke builder directly without reading spec
2. Add detailed instructions to prompt
3. Bypass routing logic
```

**Command responsibilities:**
- Read specs
- Detect execution strategy
- Route to appropriate agent
- Use minimal prompts (< 20 words)

**Agent responsibilities:**
- **Orchestrator:** Coordinate parallel builds
- **Builder:** Implement features
- **Validator:** Test features
- **Planner:** Design features

## Testing the Fix

After updating RAD commands, test with a parallel spec:

1. Create a test spec with parallel strategy
2. Run `/build` and watch which agent is invoked
3. Verify command reads the spec
4. Verify command outputs routing decision
5. Verify **orchestrator** is invoked (not builder)
6. Verify orchestrator announces: "📊 ORCHESTRATION COMPLETE"
7. Verify orchestrator spawns multiple Task tool calls
8. Verify you see "Spawning X parallel builders..."

## Need More Help?

Check these files:
- `commands/README.md` - Command best practices
- `commands/build.md` - Build command documentation
- `agents/builder.md` - Builder agent logic
- `LOOP-MECHANISM.md` - Builder-validator feedback loop

Or file an issue at: https://github.com/anthropics/claude-code/issues
