---
name: module-stub-generator
description: Generates mock/stub implementations of module interfaces to unblock parallel development when modules have dependencies. Use when module dependencies create build blockers OR when module needs to build before its dependency is ready OR when testing modules in isolation.
allowed-tools: Read, Write
---

# Module Stub Generator

You generate mock/stub implementations of module interfaces that allow dependent modules to build and test before the real implementation is ready.

## When to use
- Module D depends on Module A, but Module A isn't finished yet
- Need to unblock parallel development
- Want to test module in isolation (without real dependencies)
- Integration phase needs placeholder implementations
- Validating module against interface contract before implementation exists

## Purpose

**Problem**: Module D (auth API) depends on Module A (password validator). Module A is still being built, so Module D builder is blocked.

**Solution**: Generate stub implementation of Module A's interface. Module D builds and tests against the stub. Integration phase replaces stub with real Module A.

**Benefits**:
- True parallel development - no blocking
- Module D can be built and tested independently
- Clear separation between interface and implementation
- Easy to replace stub with real implementation
- Faster iteration - don't wait for dependencies

## Stub types

### 1. Interface-compliant stub
Implements the interface with minimal, predictable behavior.

**Use for**: Building and compiling dependent modules

### 2. Test stub (mock)
Implements interface with configurable responses for testing.

**Use for**: Unit testing dependent modules

### 3. Recording stub
Tracks calls made to it for debugging.

**Use for**: Understanding how modules interact

## Stub generation process

1. **Read interface definition**: Load interface from `interfaces/` directory
2. **Extract methods/functions**: Identify all methods that need implementation
3. **Generate stub code**: Create minimal implementation for each method
4. **Add documentation**: Mark clearly as stub, not production code
5. **Make configurable**: Allow test overrides if needed
6. **Save stub file**: Write to appropriate location

## Examples

### TypeScript Stub Example

**Original Interface**:
```typescript
// interfaces/password-validator.interface.ts
export interface ValidationResult {
  valid: boolean;
  errors: string[];
}

export interface PasswordValidator {
  validate(password: string): ValidationResult;
  hashPassword(password: string): Promise<string>;
  verifyPassword(password: string, hash: string): Promise<boolean>;
}
```

**Generated Stub**:
```typescript
// stubs/password-validator.stub.ts

/**
 * STUB IMPLEMENTATION
 *
 * This is a mock implementation of PasswordValidator interface.
 * Used for testing and parallel development.
 * DO NOT use in production - replace with real implementation.
 *
 * Generated: 2025-01-25
 * Interface: interfaces/password-validator.interface.ts
 */

import { PasswordValidator, ValidationResult } from '../interfaces/password-validator.interface';

export class PasswordValidatorStub implements PasswordValidator {
  // Configurable behavior for testing
  private shouldValidate: boolean = true;
  private validationErrors: string[] = [];

  /**
   * Stub: Always returns configurable validation result.
   * Real implementation would check password strength.
   */
  validate(password: string): ValidationResult {
    // Stub logic: simple length check only
    if (password.length < 8) {
      return {
        valid: false,
        errors: ['Password too short (stub validation)']
      };
    }

    return {
      valid: this.shouldValidate,
      errors: this.validationErrors
    };
  }

  /**
   * Stub: Returns fake hash.
   * Real implementation would use bcrypt.
   */
  async hashPassword(password: string): Promise<string> {
    // Stub: Return predictable fake hash
    return `stub_hash_${password}`;
  }

  /**
   * Stub: Simple string comparison.
   * Real implementation would use bcrypt.compare.
   */
  async verifyPassword(password: string, hash: string): Promise<boolean> {
    // Stub: Check if hash matches our fake format
    return hash === `stub_hash_${password}`;
  }

  // Test helpers (not in interface)

  /**
   * Test helper: Configure stub to fail validation.
   */
  setShouldValidate(valid: boolean, errors: string[] = []): void {
    this.shouldValidate = valid;
    this.validationErrors = errors;
  }

  /**
   * Test helper: Reset stub to default behavior.
   */
  reset(): void {
    this.shouldValidate = true;
    this.validationErrors = [];
  }
}

// Default export for convenience
export default new PasswordValidatorStub();
```

**Usage by Module D**:
```typescript
// Module D can now build and test without waiting for Module A

import { PasswordValidatorStub } from '../stubs/password-validator.stub';

// Use stub during development
const validator = new PasswordValidatorStub();

// Test Module D logic
class AuthController {
  constructor(private passwordValidator: PasswordValidator) {}

  async login(username: string, password: string) {
    // Works with stub OR real implementation
    const validation = this.passwordValidator.validate(password);
    if (!validation.valid) {
      throw new Error('Invalid password');
    }

    const hash = await this.passwordValidator.hashPassword(password);
    // ... rest of logic
  }
}

// Unit test using stub
describe('AuthController', () => {
  it('should reject invalid password', () => {
    const stub = new PasswordValidatorStub();
    stub.setShouldValidate(false, ['Password too weak']);

    const controller = new AuthController(stub);
    expect(() => controller.login('user', 'bad')).toThrow('Invalid password');
  });
});
```

### Python Stub Example

**Original Interface**:
```python
# interfaces/token_manager.py
from typing import Protocol
from datetime import timedelta

class TokenManager(Protocol):
    def generate_token(self, user_id: str, expires_in: timedelta) -> str: ...
    def verify_token(self, token: str) -> dict: ...
    def refresh_token(self, token: str) -> str: ...
```

**Generated Stub**:
```python
# stubs/token_manager_stub.py

"""
STUB IMPLEMENTATION

This is a mock implementation of TokenManager protocol.
Used for testing and parallel development.
DO NOT use in production - replace with real implementation.

Generated: 2025-01-25
Interface: interfaces/token_manager.py
"""

from typing import Dict
from datetime import timedelta, datetime

class TokenManagerStub:
    """Stub implementation of TokenManager protocol."""

    def __init__(self):
        # Track tokens for stub verification
        self._tokens: Dict[str, dict] = {}
        # Configurable behavior
        self._should_verify = True

    def generate_token(
        self,
        user_id: str,
        expires_in: timedelta = timedelta(hours=24)
    ) -> str:
        """
        Stub: Generates fake token.
        Real implementation would create JWT.
        """
        # Stub logic: create predictable token
        token = f"stub_token_{user_id}_{int(datetime.now().timestamp())}"

        # Store for verification
        self._tokens[token] = {
            "user_id": user_id,
            "exp": datetime.now() + expires_in
        }

        return token

    def verify_token(self, token: str) -> dict:
        """
        Stub: Verifies against stored tokens.
        Real implementation would verify JWT signature.
        """
        if not self._should_verify:
            raise InvalidTokenError("Stub configured to fail")

        if token not in self._tokens:
            raise InvalidTokenError("Unknown token (stub)")

        payload = self._tokens[token]

        # Check expiration
        if datetime.now() > payload["exp"]:
            raise InvalidTokenError("Token expired (stub)")

        return payload

    def refresh_token(self, token: str) -> str:
        """
        Stub: Creates new token from old one.
        Real implementation would verify and re-issue JWT.
        """
        payload = self.verify_token(token)
        return self.generate_token(payload["user_id"])

    # Test helpers

    def set_should_verify(self, should_verify: bool):
        """Test helper: Configure stub to fail verification."""
        self._should_verify = should_verify

    def reset(self):
        """Test helper: Reset stub to default state."""
        self._tokens.clear()
        self._should_verify = True


class InvalidTokenError(Exception):
    """Raised when token is invalid."""
    pass


# Default instance
default_stub = TokenManagerStub()
```

**Usage by Module D**:
```python
# Module D can build and test immediately

from stubs.token_manager_stub import TokenManagerStub, InvalidTokenError

class AuthAPI:
    def __init__(self, token_manager):
        self.token_manager = token_manager

    def login(self, user_id: str) -> dict:
        # Works with stub OR real implementation
        token = self.token_manager.generate_token(user_id)
        return {"access_token": token}

# Unit test using stub
def test_login():
    stub = TokenManagerStub()
    api = AuthAPI(stub)

    result = api.login("user123")
    assert "access_token" in result

    # Verify token works
    payload = stub.verify_token(result["access_token"])
    assert payload["user_id"] == "user123"
```

### Stub Features

**1. Minimal but functional**:
```python
# Bad: Too complex stub
def validate(self, data):
    # Implementing full business logic in stub - NO!
    if data.get('email'):
        if '@' not in data['email']:
            return False
        domain = data['email'].split('@')[1]
        if not is_valid_domain(domain):  # Complex logic
            return False
    # ... 50 more lines

# Good: Simple, predictable stub
def validate(self, data):
    # Stub: Just check for required fields
    return 'email' in data and 'name' in data
```

**2. Configurable for testing**:
```typescript
// Stub allows test configuration
const stub = new UserServiceStub();

// Test success case
stub.setWillSucceed(true);
await api.createUser(data);  // Works

// Test failure case
stub.setWillSucceed(false);
await api.createUser(data);  // Throws error
```

**3. Clearly marked as stub**:
```python
"""
⚠️ STUB IMPLEMENTATION - NOT FOR PRODUCTION ⚠️

This is a mock for parallel development.
Replace with real implementation from Module A.
"""
```

**4. Interface-compliant**:
```typescript
// Stub must implement all interface methods
class PaymentProcessorStub implements PaymentProcessor {
  // Must have ALL methods from interface
  processPayment(...): Promise<PaymentResult> { ... }
  refundPayment(...): Promise<RefundResult> { ... }
  getPaymentStatus(...): Promise<PaymentStatus> { ... }
}
```

## Stub generation strategy

### For each interface method:

1. **Return type is simple** (string, number, boolean):
   - Return a predictable default value
   - Example: `return "stub_value"`

2. **Return type is object**:
   - Return minimal valid object with required fields
   - Example: `return { id: "stub_id", status: "pending" }`

3. **Return type is array**:
   - Return empty array or array with one stub item
   - Example: `return [{ id: "stub_item_1" }]`

4. **Method throws error**:
   - Add configurable flag to control success/failure
   - Example: `if (this.shouldFail) throw new Error("Stub error")`

5. **Method is async**:
   - Return resolved promise with stub value
   - Example: `return Promise.resolve("stub_result")`

## Output structure

```
stubs/
  password-validator.stub.ts    # Stub for Module A
  token-manager.stub.ts          # Stub for Module B
  email-service.stub.ts          # Stub for Module C
  README.md                      # Usage guide
```

**README.md**:
```markdown
# Module Stubs

## Purpose
These are stub implementations for parallel development.
They allow modules to build and test before real implementations are ready.

## ⚠️ DO NOT USE IN PRODUCTION

These stubs are for development and testing only.

## How to Use

### Development
```typescript
import { PasswordValidatorStub } from './stubs/password-validator.stub';

// Use during development
const validator = new PasswordValidatorStub();
```

### Testing
```typescript
describe('MyModule', () => {
  it('should handle validation', () => {
    const stub = new PasswordValidatorStub();
    stub.setShouldValidate(false);  // Configure behavior

    // Test your module
    expect(() => myFunction(stub)).toThrow();
  });
});
```

### Integration
After real implementation is ready:
```typescript
// Replace stub with real implementation
- import { PasswordValidatorStub } from './stubs/...';
+ import { PasswordValidator } from './auth/password';

- const validator = new PasswordValidatorStub();
+ const validator = new PasswordValidator();
```

## Available Stubs
- `PasswordValidatorStub` - Mock password validation
- `TokenManagerStub` - Mock JWT token management
- `EmailServiceStub` - Mock email sending
```

## Instructions

1. **Read interface**: Load interface definition
2. **Identify methods**: List all methods/functions to implement
3. **Generate stub**: Create minimal implementation for each method
4. **Add configurability**: Include test helpers (setters, reset)
5. **Document clearly**: Mark as stub, explain usage
6. **Save file**: Write to `stubs/` directory
7. **Create README**: Explain how to use stubs

## Best practices

- Keep stubs simple - don't replicate real logic
- Make stubs configurable for testing
- Clearly mark as "DO NOT USE IN PRODUCTION"
- Return predictable, deterministic values
- Implement ALL interface methods (must be compliant)
- Add test helpers for configuration
- Include usage examples in comments

## Constraints

- Stub MUST implement complete interface
- Stub should NOT contain complex business logic
- Stub should be easily replaceable with real implementation
- Stub should clearly indicate it's not production code
- Stub should have predictable, deterministic behavior
