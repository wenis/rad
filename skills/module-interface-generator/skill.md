---
name: module-interface-generator
description: Generates interface definitions and contracts for modules that will be built in parallel. Use when planner creates parallel build plans OR before spawning parallel builders OR when modules need to work independently OR when defining integration points.
allowed-tools: Read, Write, Glob, Grep
---

# Module Interface Generator

You generate clean interface definitions that allow parallel module builders to work independently without conflicts.

## When to use
- Planner creating build plan with parallel modules
- Before spawning parallel builders (orchestration mode)
- When modules have dependencies but need to build in parallel
- When defining contracts between modules for integration
- When creating stub implementations for testing

## Purpose

**Problem**: Module A depends on Module B, but both need to build in parallel.

**Solution**: Generate Module B's interface first. Module A builds against the interface, Module B implements the interface. Integration phase connects them.

**Benefits**:
- True parallel development - no waiting
- Clear contracts prevent integration conflicts
- Builders know exactly what to implement
- Type safety across module boundaries
- Easy to stub for testing

## Interface types

### 1. TypeScript Interfaces
For TypeScript/JavaScript modules.

### 2. Python Protocols/Abstract Classes
For Python modules.

### 3. Go Interfaces
For Go modules.

### 4. API Contracts
For REST/GraphQL APIs between modules.

## Generation process

1. **Analyze module requirements**: Read build plan, understand what each module exports
2. **Identify contracts**: Determine what other modules need from this module
3. **Design interface**: Create minimal, focused interface definition
4. **Add type safety**: Include input/output types, error cases
5. **Generate code**: Write interface file in appropriate language
6. **Document usage**: Add examples and usage notes

## Output structure

For a parallel build with modules A, B, C:

```
interfaces/
  password-validator.interface.ts    # Interface for Module A
  token-manager.interface.ts         # Interface for Module B
  email-service.interface.ts         # Interface for Module C
  types.ts                          # Shared types
  README.md                         # Usage guide
```

## Examples

### TypeScript Interface Example

**Build Plan**: Module A (password validator), Module D (auth API) depends on A

**Generated Interface**:
```typescript
// interfaces/password-validator.interface.ts

/**
 * Password Validator Interface
 *
 * Module A (auth/password.py) implements this interface.
 * Module D (api/auth.py) depends on this interface.
 */

export interface ValidationResult {
  valid: boolean;
  errors: string[];
  strength?: 'weak' | 'medium' | 'strong';
}

export interface PasswordValidator {
  /**
   * Validates password against security requirements.
   *
   * @param password - The password to validate
   * @returns Validation result with errors if any
   *
   * @example
   * const result = validator.validate('MyP@ssw0rd123');
   * if (!result.valid) {
   *   console.error('Validation failed:', result.errors);
   * }
   */
  validate(password: string): ValidationResult;

  /**
   * Hashes password using bcrypt with salt.
   *
   * @param password - Plain text password
   * @returns Hashed password string
   * @throws {ValidationError} If password is too weak
   */
  hashPassword(password: string): Promise<string>;

  /**
   * Verifies password against stored hash.
   *
   * @param password - Plain text password to verify
   * @param hash - Stored password hash
   * @returns True if password matches hash
   */
  verifyPassword(password: string, hash: string): Promise<boolean>;
}

// Shared types
export class ValidationError extends Error {
  constructor(
    message: string,
    public errors: string[]
  ) {
    super(message);
    this.name = 'ValidationError';
  }
}
```

**Usage by Module D builder**:
```typescript
// Module D builder uses the interface, doesn't wait for implementation
import { PasswordValidator, ValidationResult } from '../interfaces/password-validator.interface';

// Builder implements auth endpoint assuming PasswordValidator exists
class AuthController {
  constructor(private passwordValidator: PasswordValidator) {}

  async login(username: string, password: string) {
    // Use interface methods - implementation comes later
    const validation = this.passwordValidator.validate(password);
    if (!validation.valid) {
      throw new Error('Invalid password');
    }

    const isValid = await this.passwordValidator.verifyPassword(
      password,
      storedHash
    );
    // ... rest of login logic
  }
}
```

**Implementation by Module A builder**:
```typescript
// Module A builder implements the interface
import { PasswordValidator, ValidationResult, ValidationError } from '../interfaces/password-validator.interface';
import bcrypt from 'bcrypt';

export class PasswordValidatorImpl implements PasswordValidator {
  validate(password: string): ValidationResult {
    const errors: string[] = [];

    if (password.length < 12) {
      errors.push('Password must be at least 12 characters');
    }
    if (!/[A-Z]/.test(password)) {
      errors.push('Password must contain uppercase letter');
    }
    // ... more validation

    return {
      valid: errors.length === 0,
      errors,
      strength: this.calculateStrength(password)
    };
  }

  async hashPassword(password: string): Promise<string> {
    const validation = this.validate(password);
    if (!validation.valid) {
      throw new ValidationError('Weak password', validation.errors);
    }
    return bcrypt.hash(password, 10);
  }

  async verifyPassword(password: string, hash: string): Promise<boolean> {
    return bcrypt.compare(password, hash);
  }

  private calculateStrength(password: string): 'weak' | 'medium' | 'strong' {
    // ... strength calculation
  }
}
```

### Python Protocol Example

**Build Plan**: Module B (JWT manager), Module D (auth API) depends on B

**Generated Interface**:
```python
# interfaces/token_manager.py

"""
Token Manager Interface

Module B (auth/jwt.py) implements this protocol.
Module D (api/auth.py) depends on this protocol.
"""

from typing import Protocol, Dict, Optional
from datetime import timedelta

class TokenPayload(TypedDict):
    """Token payload structure."""
    user_id: str
    email: str
    exp: int  # Expiration timestamp

class TokenManager(Protocol):
    """Interface for JWT token management."""

    def generate_token(
        self,
        user_id: str,
        email: str,
        expires_in: timedelta = timedelta(hours=24)
    ) -> str:
        """
        Generate JWT token for user.

        Args:
            user_id: Unique user identifier
            email: User email address
            expires_in: Token expiration duration

        Returns:
            Signed JWT token string

        Example:
            token = manager.generate_token('user123', 'user@example.com')
        """
        ...

    def verify_token(self, token: str) -> TokenPayload:
        """
        Verify and decode JWT token.

        Args:
            token: JWT token string

        Returns:
            Decoded token payload

        Raises:
            InvalidTokenError: If token is invalid or expired
        """
        ...

    def refresh_token(self, token: str) -> str:
        """
        Generate new token from existing valid token.

        Args:
            token: Current valid token

        Returns:
            New JWT token with extended expiration

        Raises:
            InvalidTokenError: If token is invalid or expired
        """
        ...

class InvalidTokenError(Exception):
    """Raised when token is invalid or expired."""
    pass
```

**Usage by Module D builder**:
```python
# Module D uses the protocol
from interfaces.token_manager import TokenManager, InvalidTokenError

class AuthAPI:
    def __init__(self, token_manager: TokenManager):
        self.token_manager = token_manager

    def login(self, user_id: str, email: str) -> dict:
        # Use interface - implementation comes from Module B
        token = self.token_manager.generate_token(user_id, email)
        return {"access_token": token, "token_type": "bearer"}

    def validate_request(self, token: str) -> dict:
        try:
            payload = self.token_manager.verify_token(token)
            return {"user_id": payload["user_id"]}
        except InvalidTokenError:
            raise HTTPException(401, "Invalid token")
```

### API Contract Example

**Build Plan**: Module C (email service), Module E (reset API) depends on C

**Generated Contract**:
```markdown
# Email Service API Contract

## Module: Email Service (Module C)
**Implemented by**: services/email.py
**Used by**: Module E (api/reset.py)

## Interface

### sendResetEmail()

**Purpose**: Send password reset email to user

**Signature**:
```python
async def send_reset_email(
    recipient: str,
    reset_token: str,
    expires_in: int = 3600
) -> EmailResult
```

**Parameters**:
- `recipient` (str, required): Email address to send to
  - Must be valid email format
  - Example: "user@example.com"
- `reset_token` (str, required): Password reset token
  - Unique token for this reset request
  - Example: "a1b2c3d4e5f6..."
- `expires_in` (int, optional): Token expiration in seconds
  - Default: 3600 (1 hour)
  - Range: 300 to 86400

**Returns**:
```python
class EmailResult:
    success: bool
    message_id: Optional[str]  # Email provider message ID
    error: Optional[str]        # Error message if failed
```

**Errors**:
- `InvalidEmailError`: Invalid recipient email format
- `EmailSendError`: Failed to send email (network, provider issue)
- `RateLimitError`: Too many emails sent (rate limiting)

**Example Usage**:
```python
from services.email import send_reset_email, EmailResult

result = await send_reset_email(
    recipient="user@example.com",
    reset_token="abc123",
    expires_in=3600
)

if result.success:
    print(f"Email sent: {result.message_id}")
else:
    print(f"Failed: {result.error}")
```

**Behavior**:
1. Validates email address format
2. Generates reset link with token
3. Sends email using configured provider (SendGrid, AWS SES, etc.)
4. Returns result immediately (async operation)
5. Retries up to 3 times on transient failures
6. Logs all send attempts

**Performance**:
- Expected latency: 200-500ms
- Timeout: 5 seconds
- Rate limit: 10 emails/minute per recipient

**Testing**:
- Use test mode to avoid sending real emails
- Mock implementation available for unit tests
```

## Interface design principles

### 1. Minimal and focused
Only include what dependent modules actually need.

❌ **Bad** - Too much:
```typescript
interface UserService {
  createUser(data: UserData): User;
  updateUser(id: string, data: Partial<UserData>): User;
  deleteUser(id: string): void;
  listUsers(filters: Filters): User[];
  exportUsers(format: string): string;
  importUsers(data: string): void;
  // ... 20 more methods
}
```

✅ **Good** - Focused:
```typescript
// Only what Module D needs from Module A
interface PasswordValidator {
  validate(password: string): ValidationResult;
  hashPassword(password: string): Promise<string>;
  verifyPassword(password: string, hash: string): Promise<boolean>;
}
```

### 2. Type-safe
Include explicit types for all inputs and outputs.

❌ **Bad** - No types:
```python
def process_payment(data):
    """Process a payment."""
    ...
```

✅ **Good** - Explicit types:
```python
def process_payment(
    amount: Decimal,
    currency: str,
    payment_method: PaymentMethod
) -> PaymentResult:
    """Process a payment and return result."""
    ...
```

### 3. Error handling
Define what errors can occur and when.

```typescript
interface PaymentProcessor {
  /**
   * @throws {InsufficientFundsError} When account balance too low
   * @throws {InvalidCardError} When card is invalid or expired
   * @throws {NetworkError} When payment provider unreachable
   */
  processPayment(amount: number, card: CardInfo): Promise<PaymentResult>;
}
```

### 4. Documentation
Include examples and usage notes.

```python
class DataValidator(Protocol):
    """
    Validates user input data.

    Example:
        validator = DataValidatorImpl()
        result = validator.validate(user_input)
        if not result.valid:
            return {"errors": result.errors}
    """
    def validate(self, data: dict) -> ValidationResult:
        ...
```

## Instructions

1. **Read build plan**: Understand which modules depend on each other
2. **Identify contracts**: List what each module exports to others
3. **Choose interface type**: TypeScript interface, Python protocol, API contract, etc.
4. **Design interface**: Minimal, focused, type-safe
5. **Generate code**: Write interface files in appropriate language
6. **Add documentation**: Examples, usage notes, error cases
7. **Create directory structure**: Organize interfaces logically
8. **Write README**: Explain how builders should use interfaces

## Output files

For each module that other modules depend on, generate:

1. **Interface file** (`.interface.ts`, `.py`, etc.)
   - Interface/protocol definition
   - Shared types
   - Error classes
   - Documentation

2. **README.md** (in interfaces/ directory)
   ```markdown
   # Module Interfaces

   ## Overview
   This directory contains interface definitions for parallel module development.

   ## How to Use
   - **Module builders**: Implement these interfaces
   - **Dependent builders**: Code against these interfaces
   - **Integration phase**: Wire implementations to dependents

   ## Interfaces
   - `PasswordValidator`: Password validation and hashing
   - `TokenManager`: JWT token management
   - `EmailService`: Email sending

   ## Example
   [Show example of implementing and using an interface]
   ```

3. **Shared types file** (`types.ts`, `types.py`)
   - Common data structures
   - Enums
   - Type aliases

## Best practices

- Keep interfaces stable - changes break parallel work
- Design interfaces before building starts
- One interface per module responsibility
- Include all necessary types in interface file
- Document error cases explicitly
- Provide usage examples
- Use semantic versioning if interfaces change
- Review interfaces with team before parallel build starts

## Constraints

- Interfaces must be language-appropriate (TypeScript interfaces, Python protocols, etc.)
- Must include all necessary imports/dependencies
- Should be minimal - only what's needed
- Must be backward compatible once parallel building starts
- Should follow project coding standards
