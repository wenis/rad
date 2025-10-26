---
name: module-interface-generator
description: Generates TypeScript interfaces, Python Protocols, Go interfaces, or API contracts for modules to enable parallel development without blocking on dependencies. Creates minimal, type-safe interface definitions with documentation, shared types, and error classes. Allows Module A to build against Module B's interface before B is implemented. Use when planner creates parallel build plans, before spawning parallel builders, when modules have cross-dependencies, defining integration points, or enabling independent module development.
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

For complete Python Protocol examples including TokenManager interface, implementation, and usage patterns, see:
- **`examples/python-protocol.md`** - Full Python Protocol pattern with TypedDict and error handling

### API Contract Example

For detailed API contract examples including complete specifications, OpenAPI schemas, and when to use contracts vs code interfaces, see:
- **`examples/api-contract.md`** - Email service contract with full documentation and OpenAPI example

## Interface design principles

For detailed interface design principles including minimal interfaces, type safety, error handling, documentation, versioning, and dependency direction, see:
- **`examples/design-principles.md`** - Complete guide with good/bad examples and quick reference table

**Key principles**:
1. **Minimal and focused** - Only what's needed
2. **Type-safe** - Explicit types for all inputs/outputs
3. **Error handling** - Document all error cases
4. **Documentation** - Include examples and usage notes
5. **Versioning** - Plan for interface changes
6. **Separation of concerns** - One interface per responsibility
7. **Dependency direction** - Depend on abstractions

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
