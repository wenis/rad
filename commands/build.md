---
name: build
description: Implements features from specs. For parallel builds (multiple modules), orchestrates builders directly using Task tool. For sequential builds (single module), delegates to builder agent.
---

# Build Feature

Implement a feature from specifications, either by orchestrating parallel module builds or by delegating to the builder agent for sequential implementation.

**Why this command exists:** Different features require different build strategies. Complex features benefit from parallel module development (faster delivery), while simple features are built sequentially (less overhead). This command detects the strategy and executes accordingly.

## Usage

Use this command when you want to:
- Implement a feature from a spec (**Why:** Specs provide clear requirements and success criteria)
- Prototype a new idea quickly (**Why:** Rapid prototyping validates ideas before full investment)
- Refactor existing code (**Why:** Structured approach reduces regression risk)
- Fix a bug (**Why:** Systematic implementation ensures proper testing)

## How This Works

**Strategy detection:**
1. Read spec file to find "Execution Strategy" section
2. **If "Parallel: N modules"** → Command orchestrates using Task tool (**Why:** Only main conversation has Task access)
3. **If "Sequential"** → Command delegates to builder agent (**Why:** Builder handles full feature implementation efficiently)

**Why commands orchestrate (not agents):**
- Task tool is ONLY available in main conversation
- Sub-agents spawned via Task cannot spawn other sub-agents
- Therefore: Commands must handle parallel orchestration directly

## Instructions

**Follow these steps in order:**

### Step 1: Determine the Build Strategy

**If user provides a spec path:**

1. Read the spec file
2. Find the "Build Plan" or "Execution Strategy" section
3. Check the strategy:
   - If says "Parallel: N modules in M phases" → Go to Step 2a (Orchestrator)
   - If says "Sequential: Single builder" → Go to Step 2b (Builder)
   - If missing or unclear → Default to Step 2b (Builder)

**If no spec provided (quick prototype):**

- Go directly to Step 2b (Builder)
- Builder will implement without formal spec

### Step 2a: Orchestrate Parallel Build (Parallel Strategy)

**YOU orchestrate directly using Task tool. Do NOT delegate to an orchestrator agent.**

**Why you orchestrate (not an agent):** Sub-agents spawned via Task cannot spawn other sub-agents. Only the main conversation (this command) has Task tool access.

1. **Parse the build plan from the spec:**
   - Identify all phases (**Why:** Phases have dependencies; must complete in order)
   - Identify modules per phase (**Why:** Modules in same phase can build in parallel)
   - Note dependencies (**Why:** Integration depends on understanding module relationships)

2. **Announce your orchestration plan to the user:**

**Why announce:** Users need visibility into parallel execution. Without announcement, builds appear silent for long periods, causing confusion.
   ```
   📋 PARALLEL BUILD ORCHESTRATION

   Spec: docs/specs/{feature-name}.md
   Strategy: Parallel ({N} modules in {M} phases)

   Phase 1: {X} modules (parallel)
   - Module A: {name}
   - Module B: {name}
   - Module C: {name}

   Phase 2: {Y} modules (sequential, depends on Phase 1)
   - Module D: {name}

   Spawning {X} parallel builders for Phase 1 now...
   ```

3. **For each phase, spawn parallel builders using Task tool:**

**Why all in ONE message:** Making 3 Task calls in a SINGLE message enables true parallelism. Sequential Task calls would defeat the purpose of parallel builds.

**Why minimal prompts:** Builder agent knows the complete process. Detailed prompts add noise and can contradict the agent's instructions. Trust the agent's expertise.

   <example>
   <name>Phase 1 with 3 independent modules</name>

   For Phase 1 with 3 modules, make 3 Task tool calls in ONE message:

   Task 1:
   ```
   Tool: Task
   subagent_type: general-purpose
   description: Build Module A - Form 4 Parser
   prompt: |
     Build Module A: Form 4 Parser from SPEC-0016.

     Read docs/specs/SPEC-0016-sec-filing-content-parsing.md and implement Module A.

     Scope: Parse Form 4 XML, create app/services/parsers/form4_parser.py, create database model and migration.

     Follow spec requirements. Report when complete.
   ```

   Task 2:
   ```
   Tool: Task
   subagent_type: general-purpose
   description: Build Module B - XBRL Parser
   prompt: |
     Build Module B: XBRL Parser from SPEC-0016.

     Read docs/specs/SPEC-0016-sec-filing-content-parsing.md and implement Module B.

     Scope: Parse XBRL 10-Q/10-K documents, create app/services/parsers/xbrl_parser.py, create database model and migration.

     Follow spec requirements. Report when complete.
   ```

   Task 3:
   ```
   Tool: Task
   subagent_type: general-purpose
   description: Build Module C - 13F Parser
   prompt: |
     Build Module C: 13F Parser from SPEC-0016.

     Read docs/specs/SPEC-0016-sec-filing-content-parsing.md and implement Module C.

     Scope: Parse 13F holdings, create app/services/parsers/form13f_parser.py, create database models and migrations.

     Follow spec requirements. Report when complete.
   ```
   </example>

4. **Monitor builder completion and spawn validators immediately:**

**Why immediately:** Fast feedback enables builders to fix issues while context is fresh. Waiting for all builders to complete delays feedback and slows iteration.

**How to spawn validators:** Use Task tool with `validator` agent, provide module-specific scope.

5. **Handle validation loops (max 3 iterations per module):**

**Why max 3:** Balances thoroughness with efficiency. After 3 attempts, issues likely require architectural changes or user guidance.

**How loops work:** Builder fixes → Validator retests → If fail, repeat (max 3 times) → If still failing, report to user

6. **After all phases complete, spawn integration builder:**

**Why wait for all phases:** Integration wires modules together; all modules must exist and be validated first.

**Integration prompt example:**
```
Build Module D: Integration Layer from SPEC-0016.

Wire the 3 completed parser modules (Form 4, XBRL, 13F) to SEC collectors.

Read docs/specs/SPEC-0016-sec-filing-content-parsing.md Module D section.

Create integration workflows and trigger logic. Report when complete.
```

7. **Report final status to user:**

**Why report:** Users need summary of what was built, validation status, and any issues encountered.

**Output format guidance:** Begin with `📊 PARALLEL BUILD COMPLETE` header for consistency.

**Example flow:**
```
User: "Build the authentication feature"
You: [Read docs/specs/authentication.md]
You: [Find "Execution Strategy: Parallel (3 modules in 2 phases)"]
You: [Announce plan]
You: [Use Task tool 3 times in ONE message to spawn Phase 1 builders]
You: [Monitor completion, spawn validators]
You: [Handle validation loops]
You: [Spawn integration builder]
You: [Report complete]
```

### Step 2b: Invoke Builder Agent (Sequential Strategy)

**Use this EXACT minimal prompt:**

```
Read the spec at docs/specs/{feature-name}.md and implement the feature.
```

**OR, if no spec:**

```
Implement {brief feature description from user}.
```

**That's it. Nothing else.**

The builder will:
- Read the spec
- Implement the feature
- Run basic smoke tests
- Report what was built

**Example:**
```
User: "Build the user profile page"
You: [Read docs/specs/user-profile.md]
You: [Find "Execution Strategy: Sequential"]
You: [Invoke builder]

Prompt: "Read the spec at docs/specs/user-profile.md and implement the feature."
```

### Step 3: Monitor and Report

After invoking the agent:

1. **Wait for completion**
2. **Review the output** (files created, what was built)
3. **Suggest next step** to user: "Ready for validation - run `/validate`"

---

## 🚨 CRITICAL: Minimal Prompts Only

**DO NOT add any of these to your agent prompts:**

- ❌ "You are the [agent] tasked with..." (no preambles)
- ❌ "## Current Context" sections (agents discover context themselves)
- ❌ "## Implementation Requirements" (requirements are in the spec)
- ❌ "## Guidelines" (agents know best practices)
- ❌ "## What to Return" (agents have their own output formats)
- ❌ Numbered implementation steps (agents plan their own approach)
- ❌ Interpretation of spec contents (let agents read specs directly)

**Enforcement Checklist:**

Before invoking an agent, verify:
- [ ] Prompt is < 20 words
- [ ] Prompt only contains: "Orchestrate..." OR "Read the spec at...and implement..."
- [ ] NO additional context, guidelines, or instructions

**If your prompt fails ANY check → DELETE IT and use the exact template above.**

---

## Validation Report Fixes

**If this is a fix iteration** (validator found issues):

1. Tell builder to read the validation report:
   ```
   Read validation report at docs/validation/{feature}-report.md and fix all issues.
   ```

2. Builder will:
   - Read the report
   - Fix specific issues
   - Report what was fixed

3. Then run `/validate` again

---

## Examples

<example>
**Scenario: Parallel build**

User: "Build the authentication feature we planned"

<steps>
1. Read docs/specs/authentication.md
2. Find: "Execution Strategy: Parallel (4 modules in 2 phases)"
3. Invoke orchestrator:
   Prompt: "Orchestrate the build for docs/specs/authentication.md"
4. Orchestrator spawns 4 builders, coordinates validation, handles integration
5. Report to user: "Feature complete. Ready for validation."
</steps>
</example>

<example>
**Scenario: Sequential build**

User: "Build the user profile page"

<steps>
1. Read docs/specs/user-profile.md
2. Find: "Execution Strategy: Sequential"
3. Invoke builder:
   Prompt: "Read the spec at docs/specs/user-profile.md and implement the feature."
4. Builder implements feature directly
5. Report to user: "Feature complete. Ready for validation."
</steps>
</example>

<example>
**Scenario: No spec (quick prototype)**

User: "Quickly prototype a settings page with dark mode toggle"

<steps>
1. No spec exists
2. Invoke builder directly:
   Prompt: "Implement a settings page with dark mode toggle."
3. Builder creates quick prototype
4. Report to user: "Prototype complete. Test it out."
</steps>
</example>

<example>
**Scenario: Fix validation issues**

User: "Fix the validation issues"

<steps>
1. Find validation report at docs/validation/auth-feature-report.md
2. Invoke builder:
   Prompt: "Read validation report at docs/validation/auth-feature-report.md and fix all issues."
3. Builder fixes issues
4. Report to user: "Issues fixed. Run `/validate` again."
</steps>
</example>

---

## Troubleshooting

**Problem: Not sure if build is parallel or sequential**

**Solution:** Read the spec. Look for "Execution Strategy" section. If unclear or missing, default to sequential (use builder).

**Problem: Orchestrator not spawning multiple builders**

**Solution:** Check that:
1. Spec says "Parallel" in execution strategy
2. You invoked orchestrator (not builder)
3. Your prompt was minimal (< 20 words)

**Problem: Build taking too long**

**Solution:**
- For parallel builds: Check orchestrator progress dashboard at `docs/progress/{feature}-build-progress.md`
- For sequential builds: Builder is working - wait for completion

---

## Next Steps

After building:
1. **Review code** with user if needed
2. **Run `/validate`** to test thoroughly
3. **If validation fails:** Run `/build` again (builder will read validation report)
4. **If validation passes:** Ready for deployment or `/rapid-dev` for full cycle

---

## Decision Flow Chart

```
User requests build
    ↓
Spec exists?
    ├─ No → Invoke builder (prototype mode)
    └─ Yes → Read spec
        ↓
    Find "Execution Strategy"
        ↓
    Strategy says "Parallel"?
        ├─ Yes → Invoke orchestrator
        │         ↓
        │    Orchestrator spawns multiple builders
        │    Monitors progress
        │    Coordinates validation
        │    Handles integration
        │         ↓
        │    Feature complete
        │
        └─ No (Sequential) → Invoke builder
                  ↓
             Builder implements directly
                  ↓
             Feature complete
```
