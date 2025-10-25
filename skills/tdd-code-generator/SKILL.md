---
name: tdd-code-generator
description: Generates code using strict TDD methodology - write failing tests first, implement minimal passing code, then refactor. Use when user explicitly requests TDD approach OR when building critical/security-sensitive features OR when spec requires test-driven development.
allowed-tools: Read, Edit, Write, Bash
---

# TDD Code Generator

You follow the classic Test-Driven Development (TDD) cycle: Red → Green → Refactor.

## When to use
- User explicitly asks for TDD approach
- Building security-critical features (auth, payments, encryption)
- Implementing complex algorithms that need verification
- Working on code that will be maintained long-term
- Spec explicitly requires TDD

## TDD Process

### 1. Red (Write failing test)
- Write a test that describes the desired behavior
- Run the test and verify it fails
- The test should fail for the right reason (not syntax errors)

### 2. Green (Make it pass)
- Write the minimum code necessary to pass the test
- No extra features, no premature optimization
- Focus on making the test pass, nothing more

### 3. Refactor (Clean up)
- Improve the code without changing behavior
- Remove duplication
- Improve naming and structure
- Run tests to ensure nothing broke

## Instructions
1. **Read the spec**: Understand requirements and acceptance criteria
2. **Write the test**: Create a failing test based on one requirement
3. **Run test**: Execute via Bash and confirm it fails
4. **Implement**: Write minimal code to pass the test
5. **Run test again**: Verify it passes
6. **Refactor**: Clean up code while keeping tests green
7. **Repeat**: Continue with next requirement

## Test structure
Follow the Arrange-Act-Assert pattern:
```python
def test_feature_name():
    # Arrange: Set up test data and conditions
    input_data = prepare_test_data()

    # Act: Execute the function being tested
    result = function_under_test(input_data)

    # Assert: Verify the outcome
    assert result == expected_value
```

## Output format
For each TDD cycle:
1. Show the failing test
2. Run it and show the error
3. Show the implementation
4. Run it again and show it passing
5. Show any refactoring done

## Examples

### Example 1: Building a password validator (Python)

**Requirement:** Password must be at least 12 characters

**Step 1 - Red (failing test):**
```python
# tests/test_password_validator.py
import pytest
from auth.password_validator import validate_password_strength

def test_password_too_short():
    result = validate_password_strength("short")
    assert result["valid"] == False
    assert "length" in result["errors"]
```

**Run test:**
```bash
pytest tests/test_password_validator.py
# FAILS: ModuleNotFoundError: No module named 'auth.password_validator'
```

**Step 2 - Green (minimal implementation):**
```python
# auth/password_validator.py
def validate_password_strength(password: str) -> dict:
    errors = []

    if len(password) < 12:
        errors.append("length")

    return {
        "valid": len(errors) == 0,
        "errors": errors
    }
```

**Run test:**
```bash
pytest tests/test_password_validator.py
# PASSES: 1 passed
```

**Step 3 - Refactor:**
```python
# auth/password_validator.py
MIN_PASSWORD_LENGTH = 12

def validate_password_strength(password: str) -> dict:
    """
    Validates password strength against security requirements.

    Args:
        password: The password to validate

    Returns:
        Dictionary with 'valid' boolean and 'errors' list
    """
    errors = []

    if len(password) < MIN_PASSWORD_LENGTH:
        errors.append("length")

    return {
        "valid": len(errors) == 0,
        "errors": errors
    }
```

**Next requirement:** Add uppercase letter check (repeat cycle)

### Example 2: Building an API endpoint (JavaScript/TypeScript)

**Requirement:** GET /users/:id returns user data

**Step 1 - Red:**
```typescript
// tests/users.test.ts
import request from 'supertest';
import { app } from '../src/app';

describe('GET /users/:id', () => {
  it('returns user data for valid id', async () => {
    const response = await request(app).get('/users/123');

    expect(response.status).toBe(200);
    expect(response.body).toHaveProperty('id', '123');
    expect(response.body).toHaveProperty('email');
  });
});
```

**Run:** FAILS (endpoint doesn't exist)

**Step 2 - Green:**
```typescript
// src/routes/users.ts
import { Router } from 'express';

const router = Router();

router.get('/users/:id', (req, res) => {
  // Hardcode for now - just make test pass
  res.json({
    id: req.params.id,
    email: 'user@example.com'
  });
});

export default router;
```

**Run:** PASSES

**Step 3 - Refactor:**
```typescript
// src/routes/users.ts
import { Router } from 'express';
import { getUserById } from '../services/userService';

const router = Router();

router.get('/users/:id', async (req, res) => {
  const user = await getUserById(req.params.id);
  res.json(user);
});

export default router;
```

## Best practices
- Write one test at a time
- Keep tests simple and focused
- Test behavior, not implementation
- Use descriptive test names
- Don't skip the refactor step
- Commit after each green test

## Anti-patterns to avoid
- Writing multiple tests before implementing
- Writing tests after the code
- Making tests pass by hardcoding values (without refactoring)
- Skipping the "run and verify it fails" step
- Testing private implementation details