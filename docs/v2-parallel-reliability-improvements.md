# V2.0 Parallel Build Reliability Improvements

**Date:** 2025-10-25
**Version:** 2.0.1
**Status:** Implemented

## Problem Statement

In the initial v2.0 parallel workflow implementation, users couldn't tell if the builder agent was executing builds in parallel or falling back to sequential mode. This created uncertainty and prevented verification that the parallel architecture was working correctly.

### Observed Issues

1. **No visibility:** Builder didn't announce whether it was orchestrating parallel builds
2. **Ambiguous detection:** Builder could choose to ignore parallel build plans
3. **Silent fallback:** Builder might build sequentially without user knowing
4. **Manual workaround needed:** Users had to manually orchestrate builders at the top level

## Solution Overview

Three coordinated improvements to guarantee reliable parallel execution:

### 1. Mandatory Mode Detection & Announcement (builder.md)

**What:** Builder agent now MUST announce its build strategy before starting work

**Location:** `/Users/wenis/Projects/rad/agents/builder.md` (new section after "Determining Your Mode")

**Key Changes:**
- New mandatory section: "MANDATORY: Mode Detection and Announcement"
- Builder must read spec and detect "Sequential" vs "Parallel" strategy
- Builder must announce strategy to user with clear format
- Critical rules enforced: MUST orchestrate if parallel detected
- Two example announcements (parallel and sequential) provided

**Example Output:**
```
📋 BUILD PLAN DETECTED

Strategy: Parallel (3 modules in 2 phases)
Mode: Orchestration

Parallel Execution Plan:
- Phase 1: 3 modules
  • Module A: Base RSS Collector
  • Module B: Ticker Extraction
  • Module C: Deduplication
- Phase 2: 3 modules (dependent on Phase 1)
  • Module D: Business Wire Collector
  • Module E: GlobeNewswire Collector
  • Module F: PR Newswire Collector

Spawning 3 parallel builders for Phase 1 now...
```

### 2. Explicit Build Strategy Detection (rapid-dev.md)

**What:** `/rapid-dev` command now detects build strategy from spec and gives explicit instructions to builder

**Location:** `/Users/wenis/Projects/rad/commands/rapid-dev.md` (Phase 2: Implementation)

**Key Changes:**
- New step 4: "Detect build strategy from spec"
- New step 5: "Invoke the builder agent with explicit mode instruction"
  - Different prompts for sequential vs parallel
  - Parallel prompt includes "Mode: Orchestration (REQUIRED)"
  - Explicitly tells builder: "DO NOT build everything yourself sequentially"
- Updated step numbering throughout document (now ends at step 14)

**Sequential Invocation:**
```
Build the [feature-name] feature from docs/specs/[feature-name].md

Mode: Direct Build (sequential)

Read the spec and implement the feature sequentially.
```

**Parallel Invocation:**
```
Build the [feature-name] feature from docs/specs/[feature-name].md

Mode: Orchestration (REQUIRED)

You MUST orchestrate parallel builders according to the build plan.
The spec specifies a parallel build strategy - you must spawn multiple
builder agents for each phase as defined in the build plan.

DO NOT build everything yourself sequentially.
```

### 3. Orchestration Verification (rapid-dev.md)

**What:** `/rapid-dev` now verifies that orchestration actually happens for parallel builds

**Location:** `/Users/wenis/Projects/rad/commands/rapid-dev.md` (Phase 2: Implementation, step 6)

**Key Changes:**
- New step 6: "Verify orchestration (for parallel builds)"
- Watch for builder's strategy announcement
- For parallel builds, expect to see: "Spawning X parallel builders for Phase 1..."
- If no parallel spawning within 1 minute → stop and investigate
- Documents common issue: "Builder may need explicit reminder to orchestrate"

## How It Works Together

### Happy Path (Parallel Build)

1. **Planner creates spec** with "Parallel: 3 modules in 2 phases"
2. **`/rapid-dev` reads spec** in Phase 2, step 4
3. **`/rapid-dev` detects** parallel strategy
4. **`/rapid-dev` invokes builder** with "Mode: Orchestration (REQUIRED)"
5. **Builder reads spec** per its mandatory first step
6. **Builder detects** "Parallel: 3 modules in 2 phases"
7. **Builder announces:**
   ```
   📋 BUILD PLAN DETECTED
   Strategy: Parallel (3 modules in 2 phases)
   Mode: Orchestration
   [lists all modules]
   Spawning 3 parallel builders for Phase 1 now...
   ```
8. **`/rapid-dev` observes** the announcement and spawning
9. **Builder spawns** 3 parallel builders using Task tool
10. **Each sub-builder** completes and reports back
11. **Integration happens** after all phases complete

### Failure Modes & Recovery

**Scenario 1: Builder doesn't announce**
- `/rapid-dev` step 5 says: "If builder doesn't announce, prompt it to do so"
- User intervention: "Please announce your build strategy"

**Scenario 2: Builder announces sequential for parallel spec**
- Mismatch detected by `/rapid-dev` step 6
- User intervention: "The spec says parallel but you announced sequential. Please re-read the spec's Build Plan section and orchestrate parallel builders."

**Scenario 3: Builder announces parallel but doesn't spawn**
- `/rapid-dev` step 6: "If you don't see parallel builder spawning within 1 minute, stop and investigate"
- User intervention: "You announced parallel mode but didn't spawn builders. Please spawn X builders for Phase 1 now."

## Benefits

### For Users
- ✅ **Visibility:** Always know if parallel execution is happening
- ✅ **Confidence:** Can verify builder understood the build plan
- ✅ **Control:** Can intervene if builder doesn't follow instructions
- ✅ **Predictability:** Clear pattern for both sequential and parallel builds

### For Workflow Reliability
- ✅ **Enforcement:** Builder MUST orchestrate when spec says parallel
- ✅ **Verification:** Multiple checkpoints to catch issues
- ✅ **Clarity:** Removes ambiguity about execution mode
- ✅ **Debuggability:** Easy to see where process breaks down

### For Future Development
- ✅ **Template:** Clear pattern for adding more build modes
- ✅ **Metrics:** Can track orchestration success rate
- ✅ **Learning:** Builds foundation for auto-optimization
- ✅ **Documentation:** Examples embedded in agent prompts

## Testing Recommendations

### Manual Testing
1. Run `/rapid-dev` on SPEC-0009 (PR Newswire Collectors)
2. Verify builder announces: "Parallel (4 modules in 2 phases)"
3. Verify builder spawns 3 parallel builders for Phase 1
4. Watch for validation happening in parallel
5. Confirm all phases complete with orchestration

### Regression Testing
1. Test sequential build (simple feature)
2. Verify builder announces: "Sequential (single builder)"
3. Confirm builder builds directly without spawning sub-agents
4. Ensure sequential path still works

### Edge Cases
1. Spec with ambiguous build plan → Builder should ask for clarification
2. Spec with no build plan → Builder should ask or default to sequential
3. Very complex parallel (5+ modules, 3+ phases) → Verify scalability

## Migration Notes

### For Existing Users
- **No breaking changes:** Sequential builds work exactly as before
- **Parallel builds improved:** Now with visibility and enforcement
- **Manual orchestration still works:** Can override if needed

### For Spec Authors (Planner Agent)
- **No changes required:** Specs with clear "Parallel: N modules" work automatically
- **Best practice:** Always include "Execution Strategy" section in Build Plan
- **Clarity helps:** More explicit module breakdown = better orchestration

## Future Enhancements

### Short-term (v2.1)
- [ ] Add progress dashboard during parallel builds (use progress-dashboard-generator skill)
- [ ] Metrics tracking: parallel vs sequential success rates
- [ ] Auto-retry if orchestration fails first time

### Medium-term (v2.2)
- [ ] Dedicated orchestrator agent (split from builder)
- [ ] Parallel build visualization
- [ ] Estimated time remaining for each phase

### Long-term (v3.0)
- [ ] Dynamic parallelization (auto-detect from code dependencies)
- [ ] Adaptive module sizing (optimize based on historical data)
- [ ] Cross-feature parallel builds (multiple features at once)

## Rollback Plan

If issues arise, rollback is simple:

1. **Revert builder.md:**
   ```bash
   git checkout HEAD~1 agents/builder.md
   ```

2. **Revert rapid-dev.md:**
   ```bash
   git checkout HEAD~1 commands/rapid-dev.md
   ```

3. **Manual orchestration still works:**
   - Users can spawn builders directly from top level
   - No dependency on these changes for functionality

## Success Metrics

Track these to measure improvement:

- **Visibility:** 100% of parallel builds announce strategy (vs 0% before)
- **Orchestration reliability:** >90% of parallel specs execute in parallel (vs uncertain before)
- **User confidence:** Fewer questions about "is it parallel?" (qualitative)
- **Time savings:** 2-3x speedup maintained for parallel builds (unchanged)

## References

- **Original issue:** User couldn't tell if parallel builders were spawned
- **Design docs:** PHILOSOPHY.md (parallel development section), LOOP-MECHANISM.md
- **Test case:** SPEC-0009 (PR Newswire Collectors) - 4 modules in 2 phases
- **Related skills:** progress-dashboard-generator, build-plan-optimizer

## Conclusion

These improvements address the core reliability gap in v2.0 parallel execution:

**Before:** Builder might or might not orchestrate parallel builds (invisible)
**After:** Builder must orchestrate and announces what it's doing (visible & enforced)

**Impact:** Users can now trust that parallel build plans will execute in parallel, with full visibility into the orchestration process.

---

**Status:** ✅ Implemented and ready for testing
**Next Steps:** Test with SPEC-0009 or create new parallel spec
**Questions:** Contact workflow team or check `/rapid-dev` documentation
