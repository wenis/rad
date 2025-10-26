# Troubleshooting Parallel Builds

## Architecture: Commands Orchestrate, Not Agents

**IMPORTANT: There is no orchestrator agent.** The `/build` command itself orchestrates parallel builds.

**Why?** The Task tool (used to spawn sub-agents) is only available in the main conversation, not to spawned sub-agents. Therefore:
- ✅ Commands can use Task (they run in main conversation)
- ❌ Agents cannot use Task (they ARE spawned via Task)

**Correct architecture:**
1. `/build` command reads spec
2. Command detects "Parallel: N modules" strategy
3. **Command itself** spawns parallel builders using Task tool
4. **Command** monitors, validates, and coordinates integration
5. **Command** reports final status

This troubleshooting guide helps diagnose parallel build issues.

## Quick Diagnosis

### Step 1: Check If Command Is Orchestrating

When you invoke `/build` on a parallel spec, the command should announce orchestration:

```
📋 PARALLEL BUILD ORCHESTRATION

Spec: docs/specs/SPEC-XXX.md
Strategy: Parallel (3 modules in 2 phases)

Phase 1: 3 modules (parallel)
- Module A: ...
- Module B: ...
- Module C: ...

Spawning 3 parallel builders for Phase 1 now...
```

**Then you should see multiple Task tool calls in ONE message.**

**What actually happened?**
- ✅ Saw announcement + multiple Task calls → Parallel orchestration working!
- ❌ Builder agent invoked instead → Command isn't detecting parallel strategy
- ❌ No Task calls after announcement → Command trying to delegate to agent (old architecture)

### Step 2: Check the Command Logic

The `/build` command should:
1. Read the spec file
2. Find "Execution Strategy" section
3. If "Parallel: N modules" detected:
   - Announce orchestration plan
   - Use Task tool to spawn parallel builders
   - Monitor and coordinate validation
4. If "Sequential" detected:
   - Invoke builder agent directly

**Did the command read the spec first?**
- ✅ YES → Command is working correctly
- ❌ NO → Command may have skipped detection logic

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
📋 PARALLEL BUILD ORCHESTRATION

Spec: docs/specs/SPEC-XXX.md
Strategy: Parallel (3 modules in 2 phases)

Phase 1: 3 modules (parallel)
- Module A: Base collector
- Module B: Parser
- Module C: Validator

Spawning 3 parallel builders for Phase 1 now...
```

**Then immediately:** Multiple Task tool invocations in a SINGLE message:
```
Task 1: Build Module A
Task 2: Build Module B
Task 3: Build Module C
```

After builders complete, the command spawns validators, handles validation loops, spawns integration builder, and reports final status.

## What NOT to See

❌ **Builder agent invoked directly** - If you see `builder(Read the spec...)` for a parallel spec, the command isn't detecting the parallel strategy

❌ **Orchestrator agent invoked** - Old architecture; orchestrator agents can't actually spawn sub-agents (they don't have Task tool access)

❌ **Sequential builds for parallel spec** - Builder using Write/Edit directly without Task spawning indicates parallel detection failed

## Architecture Explanation

**Current Architecture (Correct):**
1. `/build` command reads spec
2. Command detects "Parallel: N modules" strategy
3. **Command itself** spawns parallel builders using Task tool
4. Command monitors completion, spawns validators, handles validation loops
5. Command spawns integration builder after all phases complete
6. Command reports final status

**Why commands orchestrate, not agents:**
- Task tool is ONLY available in main conversation
- Commands run in main conversation → ✅ Can use Task
- Agents are spawned via Task → ❌ Cannot spawn other agents

**Old Architecture (Broken):**
- Command delegated to orchestrator agent
- Orchestrator tried to spawn builders via Task
- Task not available to sub-agents → orchestrator failed
- This is why orchestrator.md was removed

**Builder agent** (for sequential builds only):
- Invoked by command when "Sequential" strategy detected
- Reads spec, implements directly with Write/Edit
- Reports completion

## How to Prevent This

When writing custom commands for parallel workflows:

**✅ Correct approach (commands orchestrate):**
```markdown
1. Read the spec file
2. Find "Execution Strategy" section
3. If "Parallel" → Command uses Task tool to spawn parallel builders
4. If "Sequential" → Command invokes builder agent
5. Command monitors, validates, and coordinates
```

**❌ Wrong approach (delegating to agent):**
```markdown
1. Read spec, detect parallel
2. Invoke orchestrator agent to handle it
3. Orchestrator fails (no Task access)
4. Falls back to sequential or fails completely
```

**Key principle:** Only the main conversation (commands) can use Task tool. Agents spawned via Task cannot spawn other agents.

**Command responsibilities for parallel builds:**
- Read specs and detect strategy
- Parse build plan (phases, modules, dependencies)
- Spawn parallel builders using Task tool
- Monitor completion
- Spawn validators immediately
- Handle validation loops (max 3 iterations)
- Spawn integration builder
- Report final status

**Agent responsibilities:**
- **Builder:** Implement individual modules or complete features
- **Validator:** Test features and create validation reports
- **Planner:** Design features and create specs
- **Shipper:** Deploy and monitor features

## Testing the Fix

After updating RAD commands, test with a parallel spec:

1. Create or use an existing spec with parallel strategy
2. Run `/build docs/specs/{spec-name}.md`
3. **Verify command reads the spec** - Should see: `Read(docs/specs/...)`
4. **Verify command detects parallel** - Should output: `Strategy: Parallel (N modules in M phases)`
5. **Verify command announces orchestration** - Should see:
   ```
   📋 PARALLEL BUILD ORCHESTRATION
   Spawning N parallel builders for Phase 1 now...
   ```
6. **Verify multiple Task calls in ONE message** - Should see:
   ```
   Task 1: Build Module A
   Task 2: Build Module B
   Task 3: Build Module C
   ```
7. **Verify NO orchestrator agent invoked** - Should NOT see: `orchestrator(...)`
8. **Verify command continues monitoring** - Should see validators spawned after builders complete

## Need More Help?

Check these files:
- `commands/README.md` - Command best practices
- `commands/build.md` - Build command documentation
- `agents/builder.md` - Builder agent logic
- `LOOP-MECHANISM.md` - Builder-validator feedback loop

Or file an issue at: https://github.com/anthropics/claude-code/issues
