---
name: integration-smoke-test-generator
description: Generates fast smoke tests for integration points to quickly catch obvious wiring issues before full validation. Use after integration builder completes OR before integration validator runs OR when troubleshooting integration failures.
allowed-tools: Read, Write, Bash
---

# Integration Smoke Test Generator

You generate fast, simple smoke tests that quickly verify integration points are wired correctly before running comprehensive validation.

## When to use
- Immediately after integration builder completes
- Before running full integration validator (faster feedback)
- When integration tests are failing (isolate the issue)
- When debugging cross-module communication
- Quick verification after integration fixes

## Purpose

**Problem**: Integration validator runs comprehensive tests (10-15 minutes). If basic wiring is broken, we waste time on full test suite.

**Solution**: Run fast smoke tests first (30 seconds). Catch obvious issues immediately.

**Benefits**:
- Instant feedback (30s vs 10m)
- Catch 80% of integration issues fast
- Save time on obvious failures
- Clear attribution (know which integration point failed)
- Faster iteration in integration phase

## Smoke test types

### 1. Import/Export Smoke Test
Verify modules can import from each other.

```python
# Smoke test: Can Module D import from Module A?
def test_module_a_exports_available():
    """Verify Module A exports are importable."""
    from auth.password import validate_password, hash_password, verify_password
    assert callable(validate_password)
    assert callable(hash_password)
    assert callable(verify_password)
```

### 2. Basic Function Call Smoke Test
Verify functions can be called without errors.

```python
# Smoke test: Can Module D call Module A functions?
def test_password_validation_callable():
    """Verify password validation can be called."""
    from auth.password import validate_password

    # Simple smoke test - just verify it runs
    result = validate_password("test123")
    assert result is not None
    assert isinstance(result, dict)
    assert "valid" in result
```

### 3. Type Compatibility Smoke Test
Verify return types match expectations.

```typescript
// Smoke test: Does Module A return expected type?
test('password validator returns ValidationResult', () => {
  const result = passwordValidator.validate('test123');

  expect(result).toHaveProperty('valid');
  expect(result).toHaveProperty('errors');
  expect(typeof result.valid).toBe('boolean');
  expect(Array.isArray(result.errors)).toBe(true);
});
```

### 4. Integration Point Smoke Test
Verify connection between two modules works.

```python
# Smoke test: Module D → Module A integration
def test_auth_api_uses_password_validator():
    """Verify Auth API can use Password Validator."""
    from api.auth import AuthAPI
    from auth.password import PasswordValidator

    api = AuthAPI(PasswordValidator())

    # Smoke test: Just verify instantiation and basic call works
    assert api is not None
    # Don't test full logic - just that wiring works
```

## Generation strategy

For each integration point in build plan:

1. **Identify connection**: Module D uses Module A
2. **Generate import test**: Verify Module D can import Module A
3. **Generate call test**: Verify Module D can call Module A functions
4. **Generate type test**: Verify return types match
5. **Keep tests minimal**: Fast, not comprehensive

## Example output

```python
# tests/integration/smoke_test_integration.py
"""
Integration Smoke Tests

IMPORTANT: These are SMOKE TESTS for fast feedback.
They check basic wiring, not comprehensive functionality.
Full integration tests in test_integration_comprehensive.py

Generated: 2025-01-25
Feature: User Authentication
"""

import pytest

# ============================================================================
# Phase 1: Module Exports Smoke Tests
# ============================================================================

def test_module_a_exports_available():
    """SMOKE: Module A exports are importable."""
    from auth.password import validate_password, hash_password, verify_password
    assert all([validate_password, hash_password, verify_password])


def test_module_b_exports_available():
    """SMOKE: Module B exports are importable."""
    from auth.jwt import generate_token, verify_token, refresh_token
    assert all([generate_token, verify_token, refresh_token])


def test_module_c_exports_available():
    """SMOKE: Module C exports are importable."""
    from services.email import send_email
    assert send_email is not None


# ============================================================================
# Phase 2: Basic Function Call Smoke Tests
# ============================================================================

def test_password_validation_smoke():
    """SMOKE: Password validation is callable."""
    from auth.password import validate_password

    result = validate_password("testpassword123")
    assert result is not None
    assert "valid" in result


def test_token_generation_smoke():
    """SMOKE: Token generation is callable."""
    from auth.jwt import generate_token

    token = generate_token(user_id="test123")
    assert token is not None
    assert isinstance(token, str)
    assert len(token) > 0


def test_email_sending_smoke():
    """SMOKE: Email sending is callable (test mode)."""
    from services.email import send_email

    # Use test mode to avoid actually sending
    result = send_email(
        recipient="test@example.com",
        subject="Test",
        body="Test",
        test_mode=True
    )
    assert result is not None


# ============================================================================
# Phase 3: Integration Point Smoke Tests
# ============================================================================

def test_auth_api_can_use_password_validator():
    """SMOKE: Auth API can instantiate with Password Validator."""
    from api.auth import AuthAPI
    from auth.password import PasswordValidator

    # Just verify wiring works
    api = AuthAPI(password_validator=PasswordValidator())
    assert api is not None
    assert api.password_validator is not None


def test_auth_api_can_use_token_manager():
    """SMOKE: Auth API can instantiate with Token Manager."""
    from api.auth import AuthAPI
    from auth.jwt import TokenManager

    api = AuthAPI(token_manager=TokenManager())
    assert api is not None
    assert api.token_manager is not None


def test_reset_api_can_use_password_validator():
    """SMOKE: Reset API can instantiate with Password Validator."""
    from api.reset import PasswordResetAPI
    from auth.password import PasswordValidator

    api = PasswordResetAPI(password_validator=PasswordValidator())
    assert api is not None


def test_reset_api_can_use_email_service():
    """SMOKE: Reset API can instantiate with Email Service."""
    from api.reset import PasswordResetAPI
    from services.email import EmailService

    api = PasswordResetAPI(email_service=EmailService(test_mode=True))
    assert api is not None


# ============================================================================
# Phase 4: Type Compatibility Smoke Tests
# ============================================================================

def test_password_validator_returns_correct_type():
    """SMOKE: Password validator returns ValidationResult type."""
    from auth.password import validate_password

    result = validate_password("test123")

    # Check structure matches interface
    assert isinstance(result, dict)
    assert "valid" in result
    assert "errors" in result
    assert isinstance(result["valid"], bool)
    assert isinstance(result["errors"], list)


def test_token_manager_returns_correct_type():
    """SMOKE: Token manager returns string token."""
    from auth.jwt import generate_token

    token = generate_token(user_id="test123")

    assert isinstance(token, str)
    assert len(token) > 20  # JWT tokens are long


# ============================================================================
# Phase 5: End-to-End Smoke Test
# ============================================================================

def test_complete_integration_smoke():
    """SMOKE: All modules wire together for basic flow."""
    from api.auth import AuthAPI
    from auth.password import PasswordValidator
    from auth.jwt import TokenManager

    # Create API with all dependencies
    api = AuthAPI(
        password_validator=PasswordValidator(),
        token_manager=TokenManager()
    )

    # Smoke test: Just verify it exists and is wired
    assert api is not None
    assert api.password_validator is not None
    assert api.token_manager is not None

    # Don't test full login flow - that's for comprehensive tests
    # Just verify the wiring is correct


# ============================================================================
# Test Execution Info
# ============================================================================

if __name__ == "__main__":
    pytest.main([__file__, "-v", "--tb=short"])
    print("\n✅ Smoke tests complete!")
    print("Next: Run comprehensive integration tests")
```

## Smoke test report

```markdown
# Integration Smoke Test Results

**Feature**: User Authentication
**Execution Time**: 24 seconds
**Tests Run**: 13
**Status**: ✅ All smoke tests passed

---

## Smoke Test Results

### Module Exports (3 tests)
- ✅ Module A exports available
- ✅ Module B exports available
- ✅ Module C exports available

### Basic Function Calls (3 tests)
- ✅ Password validation callable
- ✅ Token generation callable
- ✅ Email sending callable

### Integration Points (4 tests)
- ✅ Auth API ↔ Password Validator
- ✅ Auth API ↔ Token Manager
- ✅ Reset API ↔ Password Validator
- ✅ Reset API ↔ Email Service

### Type Compatibility (2 tests)
- ✅ Password validator return type correct
- ✅ Token manager return type correct

### End-to-End Smoke (1 test)
- ✅ Complete integration wiring

---

## Summary

**Status**: ✅ Integration wiring is correct

All basic integration points are working:
- Modules can import from each other ✅
- Functions are callable ✅
- Types are compatible ✅
- Dependencies are wired correctly ✅

**Next Step**: Proceed to comprehensive integration validation

**Estimated Time Saved**: ~9m 30s
(Smoke tests: 24s vs Full validation: 10m if basic wiring broken)
```

## Failed smoke test example

```markdown
# Integration Smoke Test Results

**Feature**: User Authentication
**Execution Time**: 12 seconds (stopped early)
**Tests Run**: 5 / 13
**Status**: ❌ Smoke tests failed

---

## Failed Tests

### ❌ Module A exports not available

**Test**: `test_module_a_exports_available`
**Error**:
```
ImportError: cannot import name 'hash_password' from 'auth.password'

Expected exports: validate_password, hash_password, verify_password
Actual exports: validate_password, verify_password

Missing: hash_password
```

**Fix**: Module A builder needs to add `hash_password` export

---

## Recommendations

1. ❌ **Do NOT run full integration tests** - basic wiring is broken
2. 🔧 **Fix Module A**: Add missing `hash_password` export
3. 🔄 **Re-run smoke tests**: Verify fix before comprehensive validation

**Time Saved**: ~9 minutes
(Caught issue in 12s instead of discovering after 10m of full tests)
```

## Instructions

1. **Read build plan**: Understand integration points
2. **Read interfaces**: Know what to expect
3. **Generate smoke tests**: Fast, minimal tests per integration point
4. **Run tests**: Execute immediately after integration build
5. **Report results**: Pass/fail with specific issues
6. **Recommend next step**: Full validation if pass, fixes if fail

## Test characteristics

Smoke tests should be:
- ✅ **Fast**: <30 seconds total
- ✅ **Minimal**: Test wiring, not logic
- ✅ **Clear**: Obvious what failed
- ✅ **Targeted**: One test per integration point
- ❌ **Not comprehensive**: That's the full validator's job

## Best practices

- Generate automatically from build plan
- Run immediately after integration build
- Stop on first failure (fail-fast)
- Provide specific fix guidance
- Save time by catching obvious issues early
- Don't replace comprehensive tests - complement them

## Constraints

- Must be fast (<1 minute total)
- Must test actual integration points (not isolated modules)
- Must provide clear failure messages
- Should not duplicate comprehensive tests
- Focus on wiring, not functionality
