---
name: test-coverage-analyzer
description: Analyzes test coverage reports and identifies untested code paths. Use when checking test completeness OR reviewing coverage reports OR identifying gaps in test suites OR before deployment. Supports pytest-cov, jest, vitest, go test, and similar tools.
allowed-tools: Read, Grep, Bash, Write
---

# Test Coverage Analyzer

You analyze test coverage to identify untested code and provide actionable recommendations for improving test completeness.

## When to use
- After running tests with coverage enabled
- During code review to check test completeness
- Before deployment to ensure critical paths are tested
- When investigating why coverage is low
- Setting up coverage requirements for CI/CD

## Supported coverage formats

### Python (pytest-cov, coverage.py)
- `.coverage` - Binary coverage data
- `coverage.json` - JSON format
- `coverage.xml` - XML format (Cobertura)
- Terminal output from `pytest --cov`

### JavaScript/TypeScript (jest, vitest, c8)
- `coverage/coverage-final.json` - JSON format
- `coverage/lcov.info` - LCOV format
- `coverage/clover.xml` - Clover XML
- Terminal output

### Go
- `coverage.out` - Go coverage format
- `go tool cover -html=coverage.out` output

### Other
- Generic LCOV format
- Cobertura XML
- JaCoCo XML (Java)

## Analysis workflow

1. **Locate coverage files**: Search for coverage reports
2. **Parse format**: Extract coverage data
3. **Identify gaps**: Find files/functions with low coverage
4. **Prioritize**: Rank by criticality (critical paths first)
5. **Generate report**: Markdown with actionable recommendations
6. **Suggest tests**: Specific test cases for uncovered paths

## Coverage report structure

```markdown
# Test Coverage Analysis Report

## Summary
- **Overall Coverage**: 72.5%
- **Files Covered**: 45/52 (86.5%)
- **Lines Covered**: 1,847/2,548 (72.5%)
- **Branches Covered**: 145/208 (69.7%)

## Coverage by Component

| Component | Files | Coverage | Status |
|-----------|-------|----------|--------|
| auth/ | 5 | 92% | ✅ Good |
| api/ | 12 | 78% | ⚠️ Needs improvement |
| utils/ | 8 | 65% | ❌ Critical gaps |
| models/ | 10 | 95% | ✅ Excellent |

## Critical Gaps (< 70% coverage)

### 1. utils/validation.py - 45% coverage
**Priority**: High - Used in payment processing

**Uncovered lines**: 23-34, 67-89, 102-115

**Missing test scenarios**:
- [ ] Email validation with special characters
- [ ] Phone number validation edge cases
- [ ] Credit card number validation (Luhn algorithm)
- [ ] Error handling for invalid input types

**Suggested tests**:
```python
def test_email_validation_special_chars():
    assert is_valid_email("user+tag@example.com") == True
    assert is_valid_email("user@sub.example.co.uk") == True

def test_email_validation_invalid():
    assert is_valid_email("invalid@") == False
    assert is_valid_email("@example.com") == False
```

### 2. api/payment.py - 58% coverage
**Priority**: Critical - Handles financial transactions

**Uncovered lines**: 45-67 (refund logic), 89-102 (error handling)

**Missing test scenarios**:
- [ ] Partial refund processing
- [ ] Refund failure handling
- [ ] Network timeout during payment
- [ ] Duplicate payment prevention

**Suggested tests**:
```python
def test_refund_partial_amount():
    payment = create_payment(amount=100)
    refund = process_refund(payment, amount=50)
    assert refund.status == "completed"
    assert refund.amount == 50

def test_refund_exceeds_payment():
    payment = create_payment(amount=100)
    with pytest.raises(InvalidRefundError):
        process_refund(payment, amount=150)
```

## Files Below Threshold

Files with < 80% coverage:

| File | Coverage | Uncovered Lines | Priority |
|------|----------|-----------------|----------|
| api/payment.py | 58% | 45-67, 89-102 | Critical |
| utils/validation.py | 45% | 23-34, 67-89, 102-115 | High |
| api/webhooks.py | 72% | 12-18, 34-45 | Medium |
| utils/formatting.py | 68% | 56-78 | Low |

## Untested Functions

Functions with 0% coverage:

- `api/admin.py::delete_user()` - Admin function (lines 234-256)
- `utils/migration.py::rollback_migration()` - Error recovery (lines 89-102)
- `api/export.py::export_csv()` - Export feature (lines 45-89)

## Branch Coverage Gaps

Conditional branches not fully tested:

```python
# utils/validation.py:45
if age < 18:
    return False  # ✅ Covered
elif age > 120:
    return False  # ❌ Not covered
else:
    return True   # ✅ Covered
```

## Recommendations

### Immediate (Critical)
1. Add tests for payment refund logic (api/payment.py:45-67)
2. Test error handling in payment processing (api/payment.py:89-102)
3. Cover validation edge cases (utils/validation.py)

### Short-term (High Priority)
1. Test webhook signature validation (api/webhooks.py:12-18)
2. Add tests for admin functions (api/admin.py)
3. Test migration rollback scenarios (utils/migration.py)

### Long-term (Nice to have)
1. Increase coverage for formatting utilities
2. Test export functionality
3. Add integration tests for complete user flows

## Next Steps
1. Generate missing tests (use tdd-code-generator skill)
2. Run tests and verify coverage improves
3. Set up coverage requirements in CI (min 80%)
4. Add pre-commit hook to prevent coverage regression
```

## Parsing coverage files

### Python coverage.json
```python
import json

with open('coverage.json') as f:
    coverage = json.load(f)

for file, data in coverage['files'].items():
    executed = data['summary']['covered_lines']
    total = data['summary']['num_statements']
    percent = (executed / total) * 100 if total > 0 else 0

    if percent < 80:
        missing = data['missing_lines']
        print(f"{file}: {percent:.1f}% - Missing: {missing}")
```

### JavaScript coverage-final.json
```javascript
const coverage = require('./coverage/coverage-final.json');

Object.entries(coverage).forEach(([file, data]) => {
  const { s: statements, b: branches } = data;

  const covered = Object.values(statements).filter(v => v > 0).length;
  const total = Object.values(statements).length;
  const percent = (covered / total) * 100;

  if (percent < 80) {
    console.log(`${file}: ${percent.toFixed(1)}%`);
  }
});
```

### Go coverage.out
```bash
# Parse coverage
go tool cover -func=coverage.out | grep -v "100.0%"

# Generate HTML report
go tool cover -html=coverage.out -o coverage.html
```

## Using Bash for analysis

### Find low coverage files
```bash
# Python: Parse pytest output
pytest --cov --cov-report=term-missing | grep -v "100%" | grep "%.py"

# JavaScript: Parse jest output
npm test -- --coverage | grep -E "^\s+\w+\.\w+" | grep -v "100"

# Extract specific coverage
grep "TOTAL" coverage.txt | awk '{print $4}'
```

### Compare coverage over time
```bash
# Save current coverage
pytest --cov --cov-report=json
cp coverage.json coverage-baseline.json

# After changes, compare
pytest --cov --cov-report=json
diff coverage-baseline.json coverage.json
```

## CI/CD integration

### GitHub Actions
```yaml
- name: Run tests with coverage
  run: pytest --cov --cov-report=json --cov-report=term

- name: Check coverage threshold
  run: |
    COVERAGE=$(python -c "import json; print(json.load(open('coverage.json'))['totals']['percent_covered'])")
    if (( $(echo "$COVERAGE < 80" | bc -l) )); then
      echo "Coverage $COVERAGE% is below 80%"
      exit 1
    fi

- name: Upload coverage report
  uses: codecov/codecov-action@v3
  with:
    files: ./coverage.xml
```

### GitLab CI
```yaml
test:
  script:
    - pytest --cov --cov-report=term --cov-report=xml
  coverage: '/TOTAL.*\s+(\d+%)$/'
  artifacts:
    reports:
      coverage_report:
        coverage_format: cobertura
        path: coverage.xml
```

## Coverage goals by code type

| Code Type | Target Coverage | Rationale |
|-----------|----------------|-----------|
| Critical paths (auth, payment) | 95%+ | Financial/security impact |
| Core business logic | 85%+ | High business value |
| API endpoints | 80%+ | User-facing functionality |
| Utilities | 75%+ | Supporting functions |
| UI components | 70%+ | Manual testing possible |
| Configuration | 60%+ | Low risk |

## Instructions

1. **Locate coverage files**: Use Grep/Glob to find coverage reports
2. **Run coverage if missing**: Execute test suite with coverage
3. **Parse coverage data**: Read JSON/XML/text reports
4. **Calculate metrics**: Overall, by component, by file
5. **Identify gaps**: Files/functions below threshold
6. **Prioritize**: Critical paths first
7. **Generate recommendations**: Specific test cases to write
8. **Write report**: Save analysis to file

## Example usage

```bash
# Python: Generate coverage
pytest --cov=src --cov-report=json --cov-report=term-missing

# Analyze with this skill
# Skill will:
# 1. Read coverage.json
# 2. Identify files < 80%
# 3. Find uncovered lines
# 4. Suggest specific tests
# 5. Generate report
```

## Output artifacts
- Coverage analysis report (Markdown)
- List of suggested test cases
- Coverage trends (if historical data available)
- CI/CD config updates (coverage thresholds)

## Best practices
- Focus on critical paths first (auth, payment, data integrity)
- Don't chase 100% coverage - diminishing returns after 85%
- Test behavior, not implementation
- Exclude test files, migrations, config from coverage
- Track coverage trends over time
- Fail CI builds when coverage drops

## Constraints
- Do NOT write tests blindly just to hit coverage targets
- Do NOT test private implementation details
- Focus on business logic and critical paths
- Coverage percentage is a metric, not a goal
- High coverage doesn't guarantee quality tests
