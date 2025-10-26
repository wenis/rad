---
name: data-validator-generator
description: Generates type-safe validation schemas for API request/response, form inputs, environment variables, and database models using Zod (TypeScript), Pydantic (Python), Joi (Node.js), or JSON Schema. Creates custom validators with error messages, async validation, and schema composition. Implements validation middleware for Express/FastAPI/NestJS. Use when building REST/GraphQL APIs, creating forms with validation, enforcing environment config schemas, validating file uploads, or ensuring data integrity across layers.
allowed-tools: Read, Write, Edit
---

# Data Validator Generator

You generate type-safe validation schemas that ensure data integrity across APIs, forms, environment variables, and databases.

## When to use
- Building API endpoints (validate request bodies)
- Creating forms with validation
- Validating environment variables at startup
- Enforcing database constraints
- Migrating from unvalidated to validated code
- Implementing complex business rules

## Recommended Libraries

### TypeScript
**Zod** - Type-safe, composable, best choice for TypeScript

### Python
**Pydantic** - Type hints-based, FastAPI integration, best choice for Python

### JavaScript
**Joi** - Most popular for vanilla JS

## Quick Start: Basic Validation

### TypeScript (Zod)

```typescript
import { z } from 'zod';

// Define schema
const userSchema = z.object({
  email: z.string().email(),
  name: z.string().min(2).max(100),
  age: z.number().int().positive().max(120),
  role: z.enum(['admin', 'user', 'guest']).default('user'),
});

// Infer TypeScript type
type User = z.infer<typeof userSchema>;

// Validate
const result = userSchema.safeParse(data);
if (result.success) {
  const user: User = result.data; // Fully typed!
} else {
  console.error(result.error.errors);
}
```

### Python (Pydantic)

```python
from pydantic import BaseModel, EmailStr, Field

class User(BaseModel):
    email: EmailStr
    name: str = Field(..., min_length=2, max_length=100)
    age: int = Field(..., gt=0, le=120)
    role: str = Field(default='user', pattern='^(admin|user|guest)$')

# Validate
try:
    user = User(email='user@example.com', name='John', age=30)
    print(user.dict())
except ValidationError as e:
    print(e.errors())
```

## Complete Implementation Guides

For detailed examples with advanced features:

**Zod (TypeScript)** → `examples/zod-typescript.md`
- Complex validations (nested objects, arrays, unions)
- Custom refinements and transforms
- API integration (Express middleware)
- Form validation (React Hook Form)
- Environment variable validation
- Testing validators

**Pydantic (Python)** → `examples/pydantic-python.md`
- Complex validations (nested models, discriminated unions)
- Custom validators and root validators
- FastAPI integration
- Settings/config validation
- Database model validation
- Testing validators

**Common Patterns** → `reference/common-patterns.md`
- Email, URL, UUID validation
- Password strength requirements
- Phone numbers, credit cards
- File uploads
- Custom error messages
- Error formatting

## Essential Validation Patterns

### Nested Objects

```typescript
// Zod
const addressSchema = z.object({
  street: z.string(),
  city: z.string(),
  zipCode: z.string().regex(/^\d{5}(-\d{4})?$/),
});

const userSchema = z.object({
  name: z.string(),
  address: addressSchema,
});
```

```python
# Pydantic
class Address(BaseModel):
    street: str
    city: str
    zip_code: str = Field(regex=r'^\d{5}(-\d{4})?$')

class User(BaseModel):
    name: str
    address: Address
```

### Arrays with Constraints

```typescript
// Zod
const tagsSchema = z.array(z.string()).min(1).max(10);
```

```python
# Pydantic
from typing import List
tags: List[str] = Field(min_items=1, max_items=10)
```

### Custom Validation

```typescript
// Zod - refinements
const passwordSchema = z.string()
  .min(8)
  .refine((val) => /[A-Z]/.test(val), 'Must contain uppercase')
  .refine((val) => /[0-9]/.test(val), 'Must contain number');
```

```python
# Pydantic - validators
class Password(BaseModel):
    value: str

    @validator('value')
    def validate_password(cls, v):
        if len(v) < 8:
            raise ValueError('must be at least 8 characters')
        if not any(c.isupper() for c in v):
            raise ValueError('must contain uppercase')
        return v
```

### Conditional Validation

```typescript
// Zod
const productSchema = z.object({
  type: z.enum(['physical', 'digital']),
  weight: z.number().optional(),
  downloadUrl: z.string().url().optional(),
}).refine(
  (data) => {
    if (data.type === 'physical') return data.weight !== undefined;
    if (data.type === 'digital') return data.downloadUrl !== undefined;
    return true;
  },
  { message: 'Physical products need weight, digital need download URL' }
);
```

```python
# Pydantic
from pydantic import root_validator

class Product(BaseModel):
    type: Literal['physical', 'digital']
    weight: Optional[float] = None
    download_url: Optional[str] = None

    @root_validator
    def check_required_fields(cls, values):
        if values['type'] == 'physical' and not values.get('weight'):
            raise ValueError('physical products require weight')
        if values['type'] == 'digital' and not values.get('download_url'):
            raise ValueError('digital products require download_url')
        return values
```

## API Integration

### Express Middleware (TypeScript)

```typescript
import { Request, Response, NextFunction } from 'express';

function validate<T extends z.ZodType>(schema: T) {
  return (req: Request, res: Response, next: NextFunction) => {
    const result = schema.safeParse(req.body);

    if (!result.success) {
      return res.status(400).json({
        error: {
          code: 'VALIDATION_ERROR',
          message: 'Invalid request data',
          details: result.error.errors.map((err) => ({
            field: err.path.join('.'),
            message: err.message,
          })),
        },
      });
    }

    req.body = result.data;
    next();
  };
}

// Usage
app.post('/users', validate(userSchema), async (req, res) => {
  const user = await createUser(req.body); // Fully typed!
  res.json(user);
});
```

### FastAPI (Python)

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class CreateUserRequest(BaseModel):
    email: EmailStr
    name: str
    age: int

@app.post('/users')
async def create_user(user: CreateUserRequest):
    # user is already validated and typed!
    db_user = await db.create_user(**user.dict())
    return db_user

# Validation errors automatically return 422
```

## Environment Variables

### TypeScript (Zod)

```typescript
import { z } from 'zod';

const envSchema = z.object({
  NODE_ENV: z.enum(['development', 'staging', 'production']),
  PORT: z.string().transform(Number).pipe(z.number().int().positive()),
  DATABASE_URL: z.string().url(),
  JWT_SECRET: z.string().min(32),
  LOG_LEVEL: z.enum(['debug', 'info', 'warn', 'error']).default('info'),
});

export const env = envSchema.parse(process.env);

// Usage: env.PORT is a number, fully typed!
```

### Python (Pydantic)

```python
from pydantic import BaseSettings

class Settings(BaseSettings):
    database_url: str
    jwt_secret: str = Field(min_length=32)
    log_level: str = Field(default='info')

    class Config:
        env_file = '.env'

settings = Settings()
```

## Error Handling

### Format Errors for Users

```typescript
// TypeScript
function formatZodError(error: z.ZodError) {
  return error.errors.map((err) => ({
    field: err.path.join('.'),
    message: err.message,
  }));
}

// Returns:
// [
//   { field: 'email', message: 'Invalid email address' },
//   { field: 'age', message: 'Must be 18 or older' }
// ]
```

```python
# Python (FastAPI does this automatically)
# Validation errors return 422 with details
```

### Custom Error Messages

```typescript
const userSchema = z.object({
  email: z.string({
    required_error: 'Email is required',
    invalid_type_error: 'Email must be a string',
  }).email('Please enter a valid email address'),

  age: z.number().min(18, 'Must be 18 or older'),
});
```

## Testing Validators

```typescript
import { describe, it, expect } from 'vitest';

describe('userSchema', () => {
  it('validates correct data', () => {
    const result = userSchema.safeParse({
      email: 'test@example.com',
      name: 'John Doe',
      age: 30,
    });

    expect(result.success).toBe(true);
  });

  it('rejects invalid email', () => {
    const result = userSchema.safeParse({
      email: 'invalid-email',
      name: 'John Doe',
      age: 30,
    });

    expect(result.success).toBe(false);
    if (!result.success) {
      expect(result.error.errors[0].path).toEqual(['email']);
    }
  });
});
```

## Best Practices

✅ **DO:**
- Validate at API boundaries (untrusted inputs)
- Use type inference (don't duplicate types)
- Provide clear, user-friendly error messages
- Validate environment variables at startup (fail fast)
- Use refinements for complex business logic
- Transform data when parsing (trim, lowercase, parse dates)
- Test your validators (edge cases, invalid data)
- Compose schemas (build complex from simple)

❌ **DON'T:**
- Trust client-side validation only (always validate server-side)
- Validate internal function calls (type system handles it)
- Use regex for everything (use specialized validators)
- Ignore validation errors
- Hardcode validation logic in multiple places
- Leak internal error details to users
- Validate outputs (trust your own code)
- Over-validate (validate boundaries, not every function)

## Common Validation Patterns

**Email**: `z.string().email()` or `EmailStr`
**URL**: `z.string().url()` or `HttpUrl`
**UUID**: `z.string().uuid()` or `UUID4`
**Date**: `z.string().datetime()` or `datetime`
**Phone**: `z.string().regex(/^\+?[1-9]\d{1,14}$/)`
**Password**: See `reference/common-patterns.md` for strength requirements
**Credit Card**: See `reference/common-patterns.md` for Luhn validation

## Instructions

1. **Identify validation needs**
   - API request/response bodies
   - Form inputs
   - Environment variables
   - Database constraints
   - File uploads

2. **Choose library**
   - TypeScript: Zod
   - Python: Pydantic
   - Vanilla JS: Joi
   - Match your stack and framework

3. **Define schemas**
   - Start with basic types
   - Add constraints (min, max, regex)
   - Build complex schemas from simple ones
   - Use type inference

4. **Add custom validation**
   - Business rules (refinements/validators)
   - Cross-field validation (root validators)
   - Conditional requirements

5. **Integrate with framework**
   - Express: Middleware
   - FastAPI: Route parameters
   - React Hook Form: Resolver
   - Validate env vars at startup

6. **Handle errors**
   - Format errors for users
   - Map to field-level errors
   - Use custom error messages
   - Don't leak internal details

7. **Test validators**
   - Valid data passes
   - Invalid data fails
   - Edge cases
   - Custom validation logic

## Constraints

- Must validate all untrusted input (API, forms, file uploads)
- Must provide user-friendly error messages (field-level, actionable)
- Must be type-safe (infer types from schemas, single source of truth)
- Must fail fast (validate early, reject invalid data immediately)
- Must not leak internal details in error messages
- Environment variables must be validated at application startup
- Must not over-validate (boundaries only, not internal functions)
- Validation schemas must be composable and reusable
- Must handle edge cases (null, undefined, empty strings, boundary values)
