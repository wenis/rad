---
name: parallel-test-coordinator
description: Coordinates parallel test execution across multiple modules, aggregates results in real-time, and optimizes test run order. Use when running tests for parallel modules OR when validator needs to test multiple components simultaneously OR when optimizing test execution speed.
allowed-tools: Bash, Read, Write
---

# Parallel Test Coordinator

You coordinate and optimize parallel test execution across multiple modules to maximize speed while providing real-time progress updates.

## When to use
- Validator agent testing multiple modules simultaneously
- Running tests across Phase 1 modules in parallel
- Optimizing test execution order for speed
- Aggregating test results from multiple test suites
- Showing real-time test progress across modules
- Re-running only failed tests efficiently

## Purpose

**Problem**: Running module tests sequentially is slow. Module A tests (3 min) + Module B tests (2 min) + Module C tests (5 min) = 10 minutes total.

**Solution**: Run all module tests in parallel. All three finish in 5 minutes (limited by slowest).

**Benefits**:
- 2-3x faster test execution
- Real-time progress across all modules
- Early detection of failures (don't wait for all tests)
- Optimized test order (run slow tests first)
- Better resource utilization

## Coordination strategies

### 1. Full Parallel Execution
Run all module tests simultaneously.

**Best for**: Few modules (2-5), sufficient CPU/memory

### 2. Batched Parallel Execution
Run tests in batches based on available resources.

**Best for**: Many modules (6+), limited resources

### 3. Priority-Based Execution
Run critical/slow tests first, fast tests later.

**Best for**: Catching failures early

## Test execution process

1. **Analyze test suites**: Identify all test files for each module
2. **Estimate duration**: Use historical data or heuristics
3. **Determine strategy**: Full parallel, batched, or priority-based
4. **Spawn test runners**: Start tests in parallel
5. **Monitor progress**: Track completion in real-time
6. **Aggregate results**: Combine results from all modules
7. **Generate report**: Unified test report with all modules

## Parallel execution patterns

### Pattern 1: pytest in parallel (Python)

```bash
# Run Module A tests in background
pytest tests/module_a/ --json-report --json-report-file=results-a.json &
PID_A=$!

# Run Module B tests in background
pytest tests/module_b/ --json-report --json-report-file=results-b.json &
PID_B=$!

# Run Module C tests in background
pytest tests/module_c/ --json-report --json-report-file=results-c.json &
PID_C=$!

# Wait for all to complete
wait $PID_A $PID_B $PID_C

# Aggregate results
python aggregate_test_results.py results-*.json
```

### Pattern 2: vitest in parallel (JavaScript)

```bash
# Vitest supports parallel by default, but we can run separate modules in parallel

# Module A tests
vitest run tests/module-a --reporter=json --outputFile=results-a.json &
PID_A=$!

# Module B tests
vitest run tests/module-b --reporter=json --outputFile=results-b.json &
PID_B=$!

# Module C tests
vitest run tests/module-c --reporter=json --outputFile=results-c.json &
PID_C=$!

# Wait for all
wait $PID_A $PID_B $PID_C

# Aggregate
node aggregate-results.js
```

### Pattern 3: go test in parallel (Go)

```bash
# Go tests can run in parallel
go test -v -json ./module-a/... > results-a.json &
go test -v -json ./module-b/... > results-b.json &
go test -v -json ./module-c/... > results-c.json &

wait

# Aggregate
go run aggregate_results.go
```

## Real-time progress monitoring

### Track running tests

```bash
# Start tests in background with monitoring
pytest tests/module_a/ &
PID_A=$!

pytest tests/module_b/ &
PID_B=$!

pytest tests/module_c/ &
PID_C=$!

# Monitor progress
while kill -0 $PID_A 2>/dev/null || kill -0 $PID_B 2>/dev/null || kill -0 $PID_C 2>/dev/null; do
  echo "Tests running..."
  echo "  Module A: $(check_progress $PID_A)"
  echo "  Module B: $(check_progress $PID_B)"
  echo "  Module C: $(check_progress $PID_C)"
  sleep 2
done

echo "All tests complete!"
```

### Progress output

```
Running tests in parallel...

Module A: ⏳ Running... (12/25 tests, 48%)
Module B: ⏳ Running... (18/20 tests, 90%)
Module C: ⏳ Running... (5/15 tests, 33%)

---

Module A: ⏳ Running... (20/25 tests, 80%)
Module B: ✅ Complete (20/20 tests, 100% passed)
Module C: ⏳ Running... (10/15 tests, 67%)

---

Module A: ✅ Complete (25/25 tests, 100% passed)
Module B: ✅ Complete (20/20 tests, 100% passed)
Module C: ⏳ Running... (14/15 tests, 93%)

---

Module A: ✅ Complete (25/25 tests, 100% passed)
Module B: ✅ Complete (20/20 tests, 100% passed)
Module C: ✅ Complete (15/15 tests, 100% passed)

All tests complete! (60/60 passed in 5m 23s)
```

## Aggregated results format

```markdown
# Parallel Test Results

**Execution Strategy**: Full Parallel
**Modules Tested**: 3
**Total Duration**: 5m 23s
**Sequential Equivalent**: ~10m (1.9x speedup)

---

## Summary

- **Total Tests**: 60
- **Passed**: 58 ✅
- **Failed**: 2 ❌
- **Skipped**: 0
- **Overall Pass Rate**: 96.7%

---

## Module Results

### ✅ Module A: Password Validator
- **Duration**: 3m 12s
- **Tests**: 25/25 passed (100%)
- **Coverage**: 94%
- **Status**: ✅ All tests passed

**Test breakdown**:
- Unit tests: 20/20 passed
- Edge cases: 5/5 passed

---

### ❌ Module B: JWT Token Manager
- **Duration**: 2m 45s
- **Tests**: 18/20 passed (90%)
- **Coverage**: 88%
- **Status**: ❌ 2 tests failed

**Test breakdown**:
- Unit tests: 18/18 passed
- Edge cases: 0/2 passed ❌

**Failed tests**:
1. `test_token_expiration` - Token not expiring correctly
2. `test_invalid_signature` - Not catching forged signatures

**Details**:
```
FAILED tests/test_jwt.py::test_token_expiration - AssertionError: Token should be expired
FAILED tests/test_jwt.py::test_invalid_signature - Expected InvalidTokenError, got None
```

---

### ✅ Module C: Email Service
- **Duration**: 5m 23s (slowest)
- **Tests**: 15/15 passed (100%)
- **Coverage**: 85%
- **Status**: ✅ All tests passed

**Test breakdown**:
- Unit tests: 10/10 passed
- Integration tests: 5/5 passed

**Note**: Slowest module (bottleneck)

---

## Timing Analysis

| Module | Sequential Time | Parallel Time | Savings |
|--------|----------------|---------------|---------|
| Module A | 3m 12s | 3m 12s | 0s |
| Module B | 2m 45s | 2m 45s | 0s |
| Module C | 5m 23s | 5m 23s | 0s |
| **Total** | **11m 20s** | **5m 23s** | **5m 57s (52%)** |

**Explanation**: Parallel execution time = time of slowest module (Module C: 5m 23s)

---

## Recommendations

### Immediate Actions
1. **Fix Module B failures**:
   - `test_token_expiration`: Check JWT exp claim handling
   - `test_invalid_signature`: Verify signature validation logic

### Performance Optimization
1. **Module C is bottleneck** (5m 23s):
   - Consider splitting into smaller test suites
   - Or: Run Module C tests in parallel internally
   - Target: Reduce to ~3m for better overall time

### Next Run
- Re-run only failed tests: `pytest tests/module_b/test_jwt.py::test_token_expiration tests/module_b/test_jwt.py::test_invalid_signature`
- Estimated time: < 30s (vs 11m for full suite)

---

## Test Output Files

- `results-module-a.json` - Detailed Module A results
- `results-module-b.json` - Detailed Module B results (with failures)
- `results-module-c.json` - Detailed Module C results
- `combined-results.json` - Aggregated results
- `coverage-combined.xml` - Combined coverage report
```

## Optimization techniques

### 1. Historical timing data

Track test durations to optimize scheduling:

```json
{
  "module_a": {
    "average_duration": 180,
    "test_count": 25
  },
  "module_b": {
    "average_duration": 165,
    "test_count": 20
  },
  "module_c": {
    "average_duration": 323,
    "test_count": 15
  }
}
```

**Strategy**: Start slowest tests first (Module C) so they don't delay completion.

### 2. Resource-aware execution

```bash
# Detect available CPUs
CPUS=$(nproc)

# Limit parallel jobs based on resources
MAX_PARALLEL=$((CPUS / 2))

# Run tests in batches
run_in_batches() {
  local batch_size=$MAX_PARALLEL
  local modules=("$@")

  for ((i=0; i<${#modules[@]}; i+=batch_size)); do
    batch=("${modules[@]:i:batch_size}")
    for module in "${batch[@]}"; do
      run_tests "$module" &
    done
    wait
  done
}
```

### 3. Fail-fast mode

Stop all tests if critical test fails:

```bash
# Start all tests
pytest tests/module_a/ &
PID_A=$!

pytest tests/module_b/ &
PID_B=$!

pytest tests/module_c/ &
PID_C=$!

# Monitor for failures
while kill -0 $PID_A 2>/dev/null || kill -0 $PID_B 2>/dev/null || kill -0 $PID_C 2>/dev/null; do
  # Check if any test failed
  if check_failed $PID_A || check_failed $PID_B || check_failed $PID_C; then
    # Kill remaining tests
    kill $PID_A $PID_B $PID_C 2>/dev/null
    echo "Critical failure detected, stopping all tests"
    exit 1
  fi
  sleep 1
done
```

## Instructions

1. **Identify test suites**: Find test files for each module
2. **Estimate duration**: Use historical data or default estimates
3. **Determine strategy**: Choose full parallel, batched, or priority-based
4. **Start tests in parallel**: Use background jobs or test runner features
5. **Monitor progress**: Track completion, show updates
6. **Aggregate results**: Combine all results into unified report
7. **Save outputs**: Individual and combined results
8. **Report**: Generate summary with timing analysis

## Tools to use

- **Bash**: Spawn parallel test processes, monitor, aggregate
- **Read**: Read individual test result files
- **Write**: Generate aggregated test report

## Best practices

- Start slowest tests first (optimize completion time)
- Monitor in real-time (don't wait until all complete to show progress)
- Save individual results (easier debugging)
- Aggregate results (unified view)
- Track timing data (improve future runs)
- Re-run only failures (save time on iteration)
- Respect resource limits (don't overwhelm system)
- Clean up background processes (kill on interrupt)

## Constraints

- Must handle test failures gracefully
- Must aggregate results accurately
- Must track timing for optimization
- Should not overwhelm system resources
- Must provide real-time progress updates
- Should enable re-running only failed tests
