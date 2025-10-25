---
name: data-validator-generator
description: Generates validation schemas for API inputs, form data, environment variables, and database constraints using Zod, Pydantic, Joi, or JSON Schema. Use when building APIs OR creating forms OR validating user input OR enforcing data integrity.
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

## Validation Libraries

### TypeScript
- **Zod** - Type-safe, composable, recommended
- **Yup** - Popular, React Hook Form integration
- **Joi** - Battle-tested, verbose
- **io-ts** - Functional programming style

### Python
- **Pydantic** - Type hints, FastAPI integration, recommended
- **Marshmallow** - Serialization + validation
- **Cerberus** - Dictionary validation
- **jsonschema** - JSON Schema standard

### JavaScript
- **Joi** - Most popular
- **Ajv** - JSON Schema, fastest
- **validator.js** - Simple string validation

## Zod (TypeScript)

### Installation
```bash
npm install zod
```

### Basic Usage

```typescript
import { z } from 'zod';

// Define schema
const userSchema = z.object({
  email: z.string().email(),
  name: z.string().min(2).max(100),
  age: z.number().int().positive().max(120),
  role: z.enum(['admin', 'user', 'guest']).default('user'),
  website: z.string().url().optional(),
});

// Infer TypeScript type
type User = z.infer<typeof userSchema>;

// Validate data
const result = userSchema.safeParse({
  email: 'user@example.com',
  name: 'John Doe',
  age: 30,
});

if (result.success) {
  const user: User = result.data;
  console.log(user);
} else {
  console.error(result.error.errors);
}
```

### Complex Validations

```typescript
// Nested objects
const addressSchema = z.object({
  street: z.string(),
  city: z.string(),
  zipCode: z.string().regex(/^\d{5}(-\d{4})?$/),
  country: z.string().length(2), // ISO country code
});

const userWithAddressSchema = z.object({
  name: z.string(),
  address: addressSchema,
});

// Arrays
const tagsSchema = z.array(z.string()).min(1).max(10);

// Unions (OR)
const idSchema = z.union([z.string().uuid(), z.number().int().positive()]);

// Discriminated unions
const eventSchema = z.discriminatedUnion('type', [
  z.object({ type: z.literal('click'), x: z.number(), y: z.number() }),
  z.object({ type: z.literal('keypress'), key: z.string() }),
]);

// Refinements (custom validation)
const passwordSchema = z
  .string()
  .min(8)
  .refine((val) => /[A-Z]/.test(val), 'Must contain uppercase')
  .refine((val) => /[a-z]/.test(val), 'Must contain lowercase')
  .refine((val) => /[0-9]/.test(val), 'Must contain number')
  .refine((val) => /[^A-Za-z0-9]/.test(val), 'Must contain special char');

// Conditional validation
const productSchema = z.object({
  type: z.enum(['physical', 'digital']),
  weight: z.number().positive().optional(),
  downloadUrl: z.string().url().optional(),
}).refine(
  (data) => {
    if (data.type === 'physical') return data.weight !== undefined;
    if (data.type === 'digital') return data.downloadUrl !== undefined;
    return true;
  },
  { message: 'Physical products need weight, digital need download URL' }
);

// Transform (parse and transform)
const dateSchema = z.string().transform((str) => new Date(str));
const trimmedString = z.string().transform((str) => str.trim());
```

### API Integration (Express)

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
  const user = await createUser(req.body); // Typed!
  res.json(user);
});
```

### Environment Variables

```typescript
// env.ts
import { z } from 'zod';

const envSchema = z.object({
  NODE_ENV: z.enum(['development', 'staging', 'production']),
  PORT: z.string().transform(Number).pipe(z.number().int().positive()),
  DATABASE_URL: z.string().url(),
  REDIS_URL: z.string().url(),
  JWT_SECRET: z.string().min(32),
  STRIPE_SECRET_KEY: z.string().startsWith('sk_'),
  LOG_LEVEL: z.enum(['debug', 'info', 'warn', 'error']).default('info'),
});

export const env = envSchema.parse(process.env);

// Usage
console.log(`Server running on port ${env.PORT}`);
// env.PORT is number, fully typed!
```

## Pydantic (Python)

### Installation
```bash
pip install pydantic
```

### Basic Usage

```python
from pydantic import BaseModel, EmailStr, Field, validator
from typing import Optional, List
from datetime import datetime

class User(BaseModel):
    email: EmailStr
    name: str = Field(..., min_length=2, max_length=100)
    age: int = Field(..., gt=0, le=120)
    role: str = Field(default='user', pattern='^(admin|user|guest)$')
    website: Optional[str] = Field(None, regex=r'^https?://')
    created_at: datetime = Field(default_factory=datetime.utcnow)

    @validator('name')
    def name_must_not_contain_numbers(cls, v):
        if any(char.isdigit() for char in v):
            raise ValueError('name cannot contain numbers')
        return v.title()  # Capitalize

# Validate
try:
    user = User(
        email='user@example.com',
        name='john doe',
        age=30,
    )
    print(user.dict())
except ValidationError as e:
    print(e.errors())
```

### Complex Validations

```python
from pydantic import root_validator

class Address(BaseModel):
    street: str
    city: str
    zip_code: str = Field(regex=r'^\d{5}(-\d{4})?$')
    country: str = Field(min_length=2, max_length=2)

class UserWithAddress(BaseModel):
    name: str
    address: Address

# Arrays
class TagsList(BaseModel):
    tags: List[str] = Field(min_items=1, max_items=10)

# Union
from typing import Union
IdType = Union[str, int]

# Discriminated union
from typing import Literal

class ClickEvent(BaseModel):
    type: Literal['click']
    x: int
    y: int

class KeypressEvent(BaseModel):
    type: Literal['keypress']
    key: str

Event = Union[ClickEvent, KeypressEvent]

# Custom validation
class Password(BaseModel):
    value: str

    @validator('value')
    def validate_password(cls, v):
        if len(v) < 8:
            raise ValueError('must be at least 8 characters')
        if not any(c.isupper() for c in v):
            raise ValueError('must contain uppercase')
        if not any(c.islower() for c in v):
            raise ValueError('must contain lowercase')
        if not any(c.isdigit() for c in v):
            raise ValueError('must contain digit')
        return v

# Conditional validation
class Product(BaseModel):
    type: Literal['physical', 'digital']
    weight: Optional[float] = None
    download_url: Optional[str] = None

    @root_validator
    def check_required_fields(cls, values):
        type_ = values.get('type')
        if type_ == 'physical' and values.get('weight') is None:
            raise ValueError('physical products require weight')
        if type_ == 'digital' and values.get('download_url') is None:
            raise ValueError('digital products require download_url')
        return values
```

### FastAPI Integration

```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel, ValidationError

app = FastAPI()

class CreateUserRequest(BaseModel):
    email: EmailStr
    name: str
    age: int

@app.post('/users')
async def create_user(user: CreateUserRequest):
    # user is already validated!
    # TypedDict with full IDE support
    db_user = await db.create_user(**user.dict())
    return db_user

# Validation errors automatically return 422 Unprocessable Entity
```

### Settings/Config

```python
# config.py
from pydantic import BaseSettings, Field, validator

class Settings(BaseSettings):
    # Environment variables
    database_url: str
    redis_url: str
    jwt_secret: str = Field(min_length=32)
    stripe_secret_key: str
    log_level: str = Field(default='info', regex='^(debug|info|warn|error)$')

    @validator('stripe_secret_key')
    def validate_stripe_key(cls, v):
        if not v.startswith('sk_'):
            raise ValueError('Invalid Stripe secret key format')
        return v

    class Config:
        env_file = '.env'
        case_sensitive = False

settings = Settings()

# Usage
print(f"Database: {settings.database_url}")
```

## Form Validation (React Hook Form + Zod)

```typescript
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';

const signupSchema = z.object({
  email: z.string().email('Invalid email address'),
  password: z.string().min(8, 'Password must be at least 8 characters'),
  confirmPassword: z.string(),
  terms: z.boolean().refine((val) => val === true, 'You must accept terms'),
}).refine((data) => data.password === data.confirmPassword, {
  message: 'Passwords must match',
  path: ['confirmPassword'],
});

type SignupForm = z.infer<typeof signupSchema>;

export function SignupForm() {
  const {
    register,
    handleSubmit,
    formState: { errors },
  } = useForm<SignupForm>({
    resolver: zodResolver(signupSchema),
  });

  const onSubmit = async (data: SignupForm) => {
    await signup(data);
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input {...register('email')} />
      {errors.email && <span>{errors.email.message}</span>}

      <input type="password" {...register('password')} />
      {errors.password && <span>{errors.password.message}</span>}

      <input type="password" {...register('confirmPassword')} />
      {errors.confirmPassword && <span>{errors.confirmPassword.message}</span>}

      <input type="checkbox" {...register('terms')} />
      {errors.terms && <span>{errors.terms.message}</span>}

      <button type="submit">Sign up</button>
    </form>
  );
}
```

## Database Validation

### TypeORM (TypeScript)

```typescript
import { Entity, Column, PrimaryGeneratedColumn } from 'typeorm';
import { Length, IsEmail, Min, Max, IsEnum } from 'class-validator';

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  @IsEmail()
  email: string;

  @Column()
  @Length(2, 100)
  name: string;

  @Column()
  @Min(0)
  @Max(120)
  age: number;

  @Column({ type: 'enum', enum: ['admin', 'user', 'guest'], default: 'user' })
  @IsEnum(['admin', 'user', 'guest'])
  role: string;
}
```

### SQLAlchemy (Python)

```python
from sqlalchemy import Column, Integer, String, CheckConstraint
from sqlalchemy.ext.declarative import declarative_base

Base = declarative_base()

class User(Base):
    __tablename__ = 'users'

    id = Column(Integer, primary_key=True)
    email = Column(String(255), unique=True, nullable=False)
    name = Column(String(100), nullable=False)
    age = Column(Integer, CheckConstraint('age > 0 AND age <= 120'))
    role = Column(String(20), default='user')

    __table_args__ = (
        CheckConstraint("role IN ('admin', 'user', 'guest')"),
    )
```

## JSON Schema

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "required": ["email", "name", "age"],
  "properties": {
    "email": {
      "type": "string",
      "format": "email"
    },
    "name": {
      "type": "string",
      "minLength": 2,
      "maxLength": 100
    },
    "age": {
      "type": "integer",
      "minimum": 1,
      "maximum": 120
    },
    "role": {
      "type": "string",
      "enum": ["admin", "user", "guest"],
      "default": "user"
    },
    "website": {
      "type": "string",
      "format": "uri"
    }
  },
  "additionalProperties": false
}
```

## Common Validation Patterns

### Email
```typescript
z.string().email()
// Python: EmailStr from pydantic
```

### URL
```typescript
z.string().url()
// Python: HttpUrl from pydantic
```

### UUID
```typescript
z.string().uuid()
// Python: UUID4 from pydantic
```

### Date/DateTime
```typescript
z.string().datetime()  // ISO 8601
z.string().transform((str) => new Date(str))
// Python: datetime from pydantic
```

### Phone Number
```typescript
z.string().regex(/^\+?[1-9]\d{1,14}$/)  // E.164 format
```

### Credit Card (Luhn)
```typescript
const luhnCheck = (num: string) => {
  let sum = 0;
  let isEven = false;
  for (let i = num.length - 1; i >= 0; i--) {
    let digit = parseInt(num[i]);
    if (isEven) {
      digit *= 2;
      if (digit > 9) digit -= 9;
    }
    sum += digit;
    isEven = !isEven;
  }
  return sum % 10 === 0;
};

const creditCardSchema = z.string()
  .regex(/^\d{13,19}$/)
  .refine(luhnCheck, 'Invalid credit card number');
```

### Password Strength
```typescript
const passwordSchema = z.string()
  .min(8, 'At least 8 characters')
  .regex(/[A-Z]/, 'Must contain uppercase')
  .regex(/[a-z]/, 'Must contain lowercase')
  .regex(/[0-9]/, 'Must contain number')
  .regex(/[^A-Za-z0-9]/, 'Must contain special character');
```

### File Upload
```typescript
const fileSchema = z.object({
  name: z.string(),
  size: z.number().max(5 * 1024 * 1024), // 5MB
  type: z.enum(['image/jpeg', 'image/png', 'image/gif']),
});
```

## Error Messages

### Custom Error Messages

```typescript
const userSchema = z.object({
  email: z.string({
    required_error: 'Email is required',
    invalid_type_error: 'Email must be a string',
  }).email('Please enter a valid email address'),

  age: z.number({
    required_error: 'Age is required',
    invalid_type_error: 'Age must be a number',
  }).min(18, 'Must be 18 or older'),
});
```

### Formatting Errors for Users

```typescript
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

  it('rejects age over 120', () => {
    const result = userSchema.safeParse({
      email: 'test@example.com',
      name: 'John Doe',
      age: 150,
    });

    expect(result.success).toBe(false);
  });
});
```

## Best Practices

✅ **DO:**
- Validate at API boundaries (inputs)
- Use type inference (don't duplicate types)
- Provide clear error messages
- Validate environment variables at startup
- Use refinements for complex business logic
- Transform data when parsing (trim, lowercase)
- Test your validators

❌ **DON'T:**
- Trust client-side validation only
- Validate outputs (trust your code)
- Use regex for everything (use specialized validators)
- Ignore validation errors
- Hardcode validation logic in multiple places
- Validate internal function calls (type system handles it)

## Instructions

1. **Identify validation needs**: API inputs, forms, env vars, database
2. **Choose library**: Zod (TS), Pydantic (Python), appropriate for stack
3. **Define schema**: Create validation schema with types
4. **Add custom rules**: Business logic, cross-field validation
5. **Integrate**: Middleware (API), resolvers (forms), startup (env)
6. **Format errors**: User-friendly error messages
7. **Test**: Unit tests for edge cases

## Constraints

- Must validate all untrusted input
- Must provide user-friendly error messages
- Must be type-safe (infer types from schema)
- Must fail fast (invalid data rejected early)
- Must not leak internal details in errors
- Environment variables must be validated at startup
