---
name: progress-dashboard-generator
description: Generates live progress dashboards showing status of parallel builds, validation, and integration. Use when orchestrating parallel builders OR showing user build progress OR tracking multi-phase execution OR reporting completion status.
allowed-tools: Write, Read
---

# Progress Dashboard Generator

You generate and update live progress dashboards that show the status of parallel build execution in real-time.

## When to use
- Builder agent (orchestration mode) tracking parallel module builds
- Showing user progress across multiple phases
- Reporting status of parallel validation
- Displaying integration progress
- Tracking builder-validator feedback loops
- Providing visibility into long-running parallel operations

## Purpose

**Problem**: User can't see what's happening during parallel builds. Multiple builders and validators running simultaneously with no visibility.

**Solution**: Generate live status dashboard showing:
- Which modules are building, validating, or completed
- Which phase is currently executing
- Progress within each phase
- Estimated time remaining
- Any failures or issues

**Benefits**:
- User knows what's happening
- Can identify slow modules (bottlenecks)
- Can see progress even for long builds
- Can intervene if something appears stuck
- Clear communication of multi-agent workflow

## Dashboard format

Progress dashboards are Markdown documents that can be continuously updated:

```markdown
# Build Progress: [Feature Name]

**Status**: 🏗️  Building
**Started**: 2:34 PM
**Elapsed**: 3m 45s
**Strategy**: Parallel (4 modules in 2 phases)

---

## Phase 1: Independent Core Modules (2/3 complete) ⏳

### ✅ Module A: Password Validation
- **Status**: Validated
- **Duration**: 2m 15s
- **Builder**: Completed at 2:36 PM
- **Validator**: ✅ 12/12 tests passed (iteration 1)
- **Files**: `auth/password.py`, `tests/test_password.py`

### ✅ Module B: JWT Token Management
- **Status**: Validated
- **Duration**: 2m 45s
- **Builder**: Completed at 2:36 PM
- **Validator**: ✅ 15/15 tests passed (iteration 1)
- **Files**: `auth/jwt.py`, `tests/test_jwt.py`

### ⏳ Module C: Email Service
- **Status**: Validating (Iteration 1)
- **Duration**: 3m 30s (in progress)
- **Builder**: ✅ Completed at 2:37 PM
- **Validator**: 🔄 Running tests... (14/18 tests passed)
- **Files**: `services/email.py`, `tests/test_email.py`
- **Est. completion**: 30s

---

## Phase 2: Integration Layer ⏸️ Waiting

### ⏸️ Module D: Auth API Endpoint
- **Status**: Waiting for Phase 1
- **Dependencies**: Module A, Module B
- **Est. start**: When Phase 1 completes

### ⏸️ Module E: Password Reset API
- **Status**: Waiting for Phase 1
- **Dependencies**: Module A, Module C
- **Est. start**: When Phase 1 completes

---

## Integration Phase ⏸️ Waiting

### ⏸️ Wire Modules Together
- **Status**: Waiting for all phases
- **Est. duration**: ~2m

---

## Summary

- **Completed**: 2/5 modules (40%)
- **In Progress**: 1 module
- **Waiting**: 2 modules + integration
- **Issues**: None
- **Est. total time**: ~12m (4m remaining)
```

## Status icons

Use consistent icons for visual clarity:

- ✅ **Completed** - Module built and validated successfully
- ⏳ **In Progress** - Currently working
- 🔄 **Testing** - Validator running tests
- ⚠️ **Warning** - Issue detected but not blocking
- ❌ **Failed** - Module failed validation (needs fixing)
- ⏸️ **Waiting** - Blocked by dependencies
- 🏗️ **Building** - Builder agent working
- 🔍 **Validating** - Validator agent working
- 🔧 **Fixing** - Builder fixing issues from validator
- 🔗 **Integrating** - Integration phase

## Update frequency

**Initial dashboard**: Created when parallel build starts

**Update triggers**:
- Builder completes a module → Update module status to "Validating"
- Validator starts → Update with test progress
- Validator completes → Update with pass/fail, move to next phase if needed
- Builder starts fixing → Update iteration number
- Phase completes → Mark phase complete, start next phase
- Integration starts → Update integration status
- Final completion → Mark entire build as done

**Real-time updates**: Dashboard should be rewritten/updated at each trigger point

## Dashboard variations

### 1. Simple Sequential Build

```markdown
# Build Progress: Simple Feature

**Status**: ✅ Completed
**Duration**: 4m 23s
**Strategy**: Sequential (Single builder)

---

## Implementation ✅

- **Builder**: Completed at 2:35 PM
- **Files**: 3 created, 2 modified

## Validation ✅

- **Tests**: 18/18 passed
- **Coverage**: 87%
- **Security**: No issues

---

**Result**: Ready for deployment ✅
```

### 2. Parallel Build with Failure

```markdown
# Build Progress: Complex Feature

**Status**: ⚠️ Issues Detected
**Elapsed**: 8m 15s

---

## Phase 1: Core Modules (3/3 complete)

### ✅ Module A: Data Validator
- **Status**: Validated
- **Tests**: ✅ 10/10 passed

### ✅ Module B: API Client
- **Status**: Validated
- **Tests**: ✅ 15/15 passed

### ❌ Module C: Database Layer
- **Status**: Failed validation (Iteration 2/3)
- **Builder**: Completed
- **Validator**: ❌ 8/12 tests passed
- **Issues**:
  - Critical: SQL injection vulnerability in query builder
  - High: Missing transaction rollback on error
- **Action**: Builder fixing issues (iteration 2)

---

## Phase 2: Integration ⏸️

Waiting for Module C to pass validation...

---

## Summary

- **Status**: Iteration 2 in progress for Module C
- **Remaining iterations**: 1 more attempt available
- **Action required**: If iteration 3 fails, will escalate to user
```

### 3. Integration Phase Progress

```markdown
# Build Progress: Feature Complete - Integration

**Status**: 🔗 Integrating
**Elapsed**: 10m 45s

---

## All Modules ✅ Complete

- ✅ Module A, B, C (Phase 1)
- ✅ Module D, E (Phase 2)

---

## Integration Phase ⏳ In Progress

### 🔗 Wire Modules Together
- **Status**: Building integration layer
- **Progress**: Creating connection points
- **Duration**: 1m 30s

### ⏸️ Integration Validation
- **Status**: Waiting for integration build
- **Est. duration**: 2m

---

## Next Steps

1. ⏳ Complete integration build (~30s)
2. ⏸️ Run integration validator
3. ⏸️ Final system validation

**Est. completion**: 3m
```

## Generated dashboard structure

```markdown
# Build Progress: [Feature Name]

**Status**: [Overall status with icon]
**Started**: [Timestamp]
**Elapsed**: [Duration]
**Strategy**: [Parallel/Sequential with details]

---

## Phase [N]: [Phase Name] ([X]/[Y] complete) [Status Icon]

### [Status Icon] Module [Name]: [Description]
- **Status**: [Current state]
- **Duration**: [Time elapsed/total]
- **Builder**: [Status and timestamp]
- **Validator**: [Status, tests passed, iteration]
- **Files**: [List of files]
- **Issues**: [Any problems]
- **Est. completion**: [If in progress]

[Repeat for each module in phase]

---

[Repeat for each phase]

---

## [Next Phase Name] [Status Icon]

[Modules waiting or in progress]

---

## Summary

- **Completed**: [X]/[Y] modules ([%])
- **In Progress**: [Count]
- **Waiting**: [Count]
- **Issues**: [Summary]
- **Est. total time**: [Estimate] ([remaining])
```

## Instructions

1. **Initialize dashboard**: When parallel build starts
   - Create initial dashboard with all phases and modules in "Waiting" state
   - Save to `docs/progress/[feature]-build-progress.md`

2. **Update on events**: When state changes
   - Read current dashboard
   - Update relevant section (module status, phase completion, etc.)
   - Rewrite dashboard file
   - Report update to user

3. **Track time**: Show elapsed and estimated time
   - Start time for each module
   - Duration when complete
   - Estimated completion for in-progress items

4. **Show dependencies**: Make clear what's blocking what
   - "Waiting for Phase 1" for Phase 2 modules
   - "Depends on Module A, B" for dependent modules

5. **Highlight issues**: Make failures and warnings visible
   - Failed tests with error count
   - Iteration number for feedback loops
   - Escalation warnings at iteration 3

6. **Final summary**: When build completes
   - Total time
   - All modules status
   - Ready for deployment or issues requiring attention

## Output location

**Primary dashboard**: `docs/progress/[feature]-build-progress.md`
- Updated continuously during build
- Permanent record after build completes

**Console updates**: Brief status updates output to user
- "Module A completed validation ✅"
- "Phase 1 complete, starting Phase 2..."
- "Integration in progress..."

## Example update flow

```
[Build starts]
→ Generate initial dashboard (all modules "Waiting")
→ Save to docs/progress/auth-build-progress.md
→ Output: "Build started: 3 modules in 2 phases"

[Module A builder completes]
→ Update dashboard: Module A status = "Validating"
→ Save dashboard
→ Output: "Module A built, starting validation..."

[Module A validator passes]
→ Update dashboard: Module A status = "✅ Validated"
→ Save dashboard
→ Output: "Module A validated ✅ (1/3 complete)"

[All Phase 1 completes]
→ Update dashboard: Phase 1 = "✅ Complete", Phase 2 = "⏳ In Progress"
→ Save dashboard
→ Output: "Phase 1 complete ✅ Starting Phase 2..."

[Integration starts]
→ Update dashboard: Integration status = "🔗 In Progress"
→ Save dashboard
→ Output: "All modules complete, integrating..."

[Build completes]
→ Update dashboard: Status = "✅ Complete"
→ Save dashboard
→ Output: "Build complete ✅ Ready for deployment (12m 34s)"
```

## Best practices

- Update dashboard immediately when state changes
- Keep status icons consistent
- Show progress within modules (test counts, file counts)
- Estimate time remaining when possible
- Highlight bottlenecks (slowest modules)
- Make failures obvious and actionable
- Include links to detailed reports for failures
- Preserve dashboard after build completes (historical record)

## Constraints

- Dashboard must be valid Markdown
- Status must accurately reflect current state
- Timestamps must be consistent
- File paths must be correct
- Estimated times should be reasonable (based on progress)
- Don't clutter dashboard with too much detail - link to detailed reports instead
