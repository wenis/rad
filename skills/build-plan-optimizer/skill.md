---
name: build-plan-optimizer
description: Uses historical build data, success rates, and performance metrics to suggest optimal module breakdowns and parallel execution strategies. Analyzes past builds to identify patterns (which modules integrate well, which cause conflicts, optimal parallelization levels). Learns from feedback to improve future build plans. Recommends module granularity, parallelization degree, and build order. Use when planner creates build plans, analyzing build performance, improving future build strategies, or optimizing parallel execution based on historical data.
allowed-tools: Read, Write, Grep, Glob
---

# Build Plan Optimizer

You analyze historical build data and use heuristics to suggest optimal module breakdowns and parallel execution strategies.

## When to use
- Planner creating build plan (suggest optimal strategy)
- After multiple builds (learn patterns)
- When builds are slower than expected
- Optimizing frequently-built feature types
- Analyzing build performance trends

## Purpose

**Problem**: Planner doesn't know which module breakdown strategies work best. Makes suboptimal decisions.

**Solution**: Learn from historical data. Track which strategies were fast, which had integration issues. Suggest proven patterns.

**Benefits**:
- Faster builds over time (learns optimal breakdowns)
- Fewer integration failures (learns what causes conflicts)
- Better time estimates (based on actual data)
- Consistent strategies for similar features

## Optimization strategies

### 1. Historical pattern matching

```json
{
  "authentication_features": {
    "optimal_strategy": "parallel_3_modules_2_phases",
    "avg_build_time": "11m 23s",
    "success_rate": 95,
    "common_breakdown": {
      "phase_1": ["password_validation", "token_management", "email_service"],
      "phase_2": ["api_endpoints", "password_reset"]
    },
    "lessons": [
      "Email service is slowest - always in Phase 1",
      "API endpoints depend on both password and JWT - Phase 2"
    ]
  },
  "crud_features": {
    "optimal_strategy": "parallel_4_modules_1_phase",
    "avg_build_time": "8m 15s",
    "success_rate": 98,
    "common_breakdown": {
      "phase_1": ["model", "repository", "api_controller", "tests"]
    }
  }
}
```

### 2. Build time estimation

```json
{
  "module_types": {
    "password_validation": {
      "avg_duration": "3m 12s",
      "complexity": "low",
      "failure_rate": "5%"
    },
    "email_service": {
      "avg_duration": "8m 23s",
      "complexity": "high",
      "failure_rate": "15%"
    },
    "api_endpoint": {
      "avg_duration": "4m 45s",
      "complexity": "medium",
      "failure_rate": "10%"
    }
  }
}
```

### 3. Bottleneck identification

Track which modules slow down builds:

```
Historical analysis (last 20 builds):
- Email services: Avg 7m 45s (slowest 35% of time)
- Database migrations: Avg 6m 12s (slowest 25% of time)
- API controllers: Avg 3m 30s (typical)

Recommendation: Always place email services in Phase 1 to avoid blocking later phases.
```

## Optimization recommendations

### For new auth feature

**Input**: User wants authentication with email/password

**Analysis**:
```
Analyzing historical data...
Found 5 similar features: "user-auth", "admin-auth", "api-auth", "oauth-integration", "password-reset"

Best performing strategy:
- Parallel (3 modules in 2 phases)
- Avg time: 11m 23s
- Success rate: 95%

Worst performing strategy:
- Sequential (1 builder)
- Avg time: 24m 10s
- Success rate: 85% (integration issues)
```

**Recommendation**:
```markdown
## Recommended Build Plan

**Strategy**: Parallel (3 modules in 2 phases)
**Estimated Time**: 11-13 minutes
**Confidence**: High (based on 5 similar features)

### Phase 1 (Parallel)
- Module A: Password Validation (est. 3m)
- Module B: JWT Token Management (est. 3m)
- Module C: Email Service (est. 8m) ⚠️ Bottleneck

### Phase 2 (Parallel)
- Module D: Auth API (est. 4m, depends on A+B)
- Module E: Password Reset (est. 4m, depends on A+C)

### Rationale
- Similar to "user-auth" build (95% match)
- Email service in Phase 1 (learned: slowest component)
- API endpoints in Phase 2 (learned: need password+JWT ready)
- Estimated 2.1x speedup vs sequential

### Risks & Mitigations
- Email service often fails validation (15% rate)
  → Mitigation: Generate comprehensive tests upfront
- JWT/Password integration occasionally has type mismatches
  → Mitigation: Generate interfaces before building
```

## Learning from builds

### After each build, record:

```json
{
  "build_id": "auth-2025-01-25",
  "feature_type": "authentication",
  "strategy": "parallel_3_modules_2_phases",
  "modules": [
    {
      "name": "password_validation",
      "phase": 1,
      "duration": "3m 15s",
      "validation_iterations": 1,
      "status": "success"
    },
    {
      "name": "email_service",
      "phase": 1,
      "duration": "8m 45s",
      "validation_iterations": 2,
      "status": "success_after_fixes"
    }
  ],
  "total_duration": "12m 34s",
  "integration_issues": ["type mismatch in user_id parameter"],
  "lessons": [
    "Email service took longer than average (8m45s vs 7m45s avg)",
    "Type mismatch in integration - should use interface-generator"
  ],
  "rating": "success"
}
```

### Analysis queries:

```
Which strategies work best for authentication features?
→ Parallel 3 modules in 2 phases (95% success, 11m avg)

Which modules are bottlenecks?
→ Email services (8m avg), Database migrations (6m avg)

Which module types have high failure rates?
→ Email (15%), External APIs (20%), Database (12%)

What causes integration failures?
→ Type mismatches (40%), Missing exports (30%), Circular deps (20%)
```

## Optimization output

```markdown
# Build Plan Optimization Analysis

**Feature**: User Authentication
**Feature Type**: Authentication (matched historical pattern)

---

## Recommended Strategy

**Approach**: Parallel (3 modules in 2 phases)
**Estimated Time**: 11-13 minutes
**Confidence**: High (95% based on 5 similar builds)
**Expected Speedup**: 2.1x faster than sequential

### Why This Strategy

Based on historical data from similar features:
1. **Success rate**: 95% (vs 85% sequential)
2. **Build time**: 11m avg (vs 24m sequential)
3. **Integration issues**: Low (avg 1.2 issues vs 3.4 sequential)

---

## Module Breakdown

### Phase 1: Independent Core Modules

**Module A: Password Validation**
- **Type**: password_validation (standard pattern)
- **Est. Duration**: 3m (based on avg of 3m 12s)
- **Complexity**: Low
- **Failure Risk**: 5%
- **Historical Notes**: Usually passes validation first try

**Module B: JWT Token Management**
- **Type**: token_management (standard pattern)
- **Est. Duration**: 3m (based on avg of 3m 8s)
- **Complexity**: Low
- **Failure Risk**: 8%
- **Historical Notes**: Occasionally has token expiration bugs

**Module C: Email Service**
- **Type**: email_service (standard pattern)
- **Est. Duration**: 8m (based on avg of 8m 23s) ⚠️ BOTTLENECK
- **Complexity**: High
- **Failure Risk**: 15%
- **Historical Notes**: Slowest module, often needs 2 validation iterations
- **Recommendation**: Start this first, use detailed tests

### Phase 2: Integration Layer

**Module D: Auth API Endpoint**
- **Dependencies**: Module A, Module B
- **Est. Duration**: 4m (based on avg of 4m 45s)
- **Complexity**: Medium
- **Historical Notes**: Type mismatches common - use interface generator

**Module E: Password Reset Flow**
- **Dependencies**: Module A, Module C
- **Est. Duration**: 4m (based on avg of 4m 10s)
- **Complexity**: Medium

---

## Risk Analysis

### High Risk Areas (from historical data)

1. **Email Service Validation** (15% failure rate)
   - Common issues: SMTP connection, rate limiting, email format
   - **Mitigation**: Use email-service template, thorough tests

2. **Type Mismatches in Integration** (40% of integration failures)
   - Common issue: user_id as string vs number
   - **Mitigation**: Generate interfaces before building

3. **Token Expiration Logic** (8% failure rate in JWT modules)
   - Common issue: Off-by-one errors in expiration check
   - **Mitigation**: Use TDD for time-sensitive logic

---

## Optimizations

### Time Optimizations

1. **Run Module C first** (within Phase 1)
   - It's the slowest - don't let it delay other modules
   - Start at t=0, other modules can catch up

2. **Use stub for Email Service** (optional)
   - If Module E can't wait for Module C (if C takes >10m)
   - Generate stub, Module E builds against it
   - Replace with real implementation in integration

3. **Generate interfaces upfront**
   - Prevents 40% of integration issues
   - Minimal time cost (30s)
   - High value

### Quality Optimizations

1. **Use TDD for JWT module** (reduces failures from 8% to 2%)
2. **Use integration-conflict-detector** before integration (catches type mismatches)
3. **Pre-generate comprehensive email tests** (reduces email failures from 15% to 8%)

---

## Alternative Strategies Considered

### Strategy 2: Sequential (Single Builder)
- **Time**: ~24 minutes
- **Success Rate**: 85%
- **Pros**: Simpler, no coordination
- **Cons**: 2x slower, more integration issues
- **Verdict**: Not recommended

### Strategy 3: Parallel (5 modules in 3 phases)
- **Time**: ~13 minutes
- **Success Rate**: 88%
- **Pros**: More granular
- **Cons**: Overhead not worth 1m savings, more complex
- **Verdict**: Over-optimized

---

## Recommendations

### For This Build
1. ✅ Use recommended parallel strategy (3 modules, 2 phases)
2. ✅ Generate interfaces before Phase 1
3. ✅ Start with Module C (email service) to avoid bottleneck
4. ✅ Use TDD for Module B (JWT)
5. ✅ Run integration-conflict-detector before integration

### For Future Builds
1. Track this build's performance
2. Update email_service avg duration based on results
3. If successful, reinforce this pattern for auth features
4. If issues found, adjust recommendations

---

## Expected Timeline

```
t=0:00  Phase 1 starts (Modules A, B, C in parallel)
t=3:00  Modules A, B complete → validation starts
t=3:30  Modules A, B validated ✅
t=8:00  Module C completes → validation starts
t=9:00  Module C validated ✅
t=9:00  Phase 2 starts (Modules D, E in parallel)
t=13:00 Modules D, E complete and validated ✅
t=13:00 Integration starts
t=15:00 Integration complete ✅

Total: ~15 minutes
```

(Estimate includes buffer for validation iterations)
```

## Instructions

1. **Load historical data**: Read past build records
2. **Match feature type**: Find similar features
3. **Analyze performance**: Best/worst strategies
4. **Generate recommendations**: Optimal breakdown
5. **Provide estimates**: Time and risk
6. **Explain rationale**: Why this strategy
7. **Track new build**: Record results for learning

## Data sources

- `docs/metrics/build-history.json` - Past build records
- `docs/metrics/module-performance.json` - Module type averages
- `docs/validation/*-report.md` - Validation issues
- `docs/metrics/integration-failures.json` - Integration problems

## Best practices

- Learn from successes AND failures
- Provide confidence levels (high/medium/low)
- Suggest alternatives with trade-offs
- Update recommendations based on new data
- Be specific with rationale
- Include risk analysis

## Constraints

- Must base recommendations on actual data (not guesses)
- Must provide confidence levels
- Should suggest alternatives
- Must explain reasoning
- Should track and learn continuously
