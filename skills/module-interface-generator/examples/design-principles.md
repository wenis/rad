# Interface Design Principles

## 1. Minimal and Focused

Only include what dependent modules actually need.

### ❌ Bad - Too much

```typescript
interface UserService {
  createUser(data: UserData): User;
  updateUser(id: string, data: Partial<UserData>): User;
  deleteUser(id: string): void;
  listUsers(filters: Filters): User[];
  exportUsers(format: string): string;
  importUsers(data: string): void;
  getUserStats(): Stats;
  getUserActivity(id: string): Activity[];
  archiveUser(id: string): void;
  restoreUser(id: string): void;
  mergeUsers(id1: string, id2: string): User;
  // ... 20 more methods
}
```

**Problems**:
- Module only needs 2-3 methods, forced to implement 30
- Changes to any method affect all dependents
- Hard to test and maintain
- Violates Interface Segregation Principle

### ✅ Good - Focused

```typescript
// Only what Module D needs from Module A
interface PasswordValidator {
  validate(password: string): ValidationResult;
  hashPassword(password: string): Promise<string>;
  verifyPassword(password: string, hash: string): Promise<boolean>;
}
```

**Benefits**:
- Module implements exactly what it needs
- Clear, focused responsibility
- Easy to test
- Can have multiple small interfaces instead of one large one

## 2. Type-Safe

Include explicit types for all inputs and outputs.

### ❌ Bad - No types

```python
def process_payment(data):
    """Process a payment."""
    ...
```

**Problems**:
- No idea what `data` should contain
- Can't catch errors at development time
- Documentation insufficient
- Runtime errors likely

### ✅ Good - Explicit types

```python
from decimal import Decimal
from typing import Literal

PaymentMethod = Literal["card", "bank_transfer", "paypal"]

class PaymentRequest:
    amount: Decimal
    currency: str
    payment_method: PaymentMethod
    customer_id: str

class PaymentResult:
    success: bool
    transaction_id: Optional[str]
    error: Optional[str]

def process_payment(request: PaymentRequest) -> PaymentResult:
    """Process a payment and return result."""
    ...
```

**Benefits**:
- IDE autocomplete works
- Type checker catches errors
- Self-documenting
- Refactoring safe

## 3. Error Handling

Define what errors can occur and when.

### ❌ Bad - No error documentation

```typescript
interface PaymentProcessor {
  processPayment(amount: number, card: CardInfo): Promise<PaymentResult>;
}
```

**Problems**:
- Caller doesn't know what can fail
- No guidance on error handling
- Runtime surprises

### ✅ Good - Explicit errors

```typescript
class InsufficientFundsError extends Error {
  constructor(
    message: string,
    public required: number,
    public available: number
  ) {
    super(message);
    this.name = 'InsufficientFundsError';
  }
}

class InvalidCardError extends Error {
  constructor(message: string, public reason: string) {
    super(message);
    this.name = 'InvalidCardError';
  }
}

class NetworkError extends Error {
  constructor(message: string, public retryable: boolean) {
    super(message);
    this.name = 'NetworkError';
  }
}

interface PaymentProcessor {
  /**
   * Process a payment.
   *
   * @throws {InsufficientFundsError} When account balance too low
   * @throws {InvalidCardError} When card is invalid or expired
   * @throws {NetworkError} When payment provider unreachable
   */
  processPayment(amount: number, card: CardInfo): Promise<PaymentResult>;
}
```

**Benefits**:
- Caller knows all failure modes
- Can handle errors appropriately
- Type-safe error handling
- Clear error semantics

## 4. Documentation

Include examples and usage notes.

### ❌ Bad - No examples

```python
class DataValidator(Protocol):
    def validate(self, data: dict) -> ValidationResult:
        ...
```

### ✅ Good - With examples

```python
class DataValidator(Protocol):
    """
    Validates user input data against schema rules.

    This interface allows Module A to validate data while Module B
    implements the actual validation logic.

    Example:
        validator = DataValidatorImpl()
        result = validator.validate({
            "email": "user@example.com",
            "age": 25
        })

        if not result.valid:
            return {"errors": result.errors}

        # Proceed with valid data

    See also:
        - ValidationResult: Return type with errors field
        - ValidationRule: How to define custom rules
    """
    def validate(self, data: dict) -> ValidationResult:
        """
        Validate input data.

        Args:
            data: Dictionary of field names to values

        Returns:
            ValidationResult with valid flag and any errors

        Example:
            result = validator.validate({"email": "invalid"})
            # result.valid == False
            # result.errors == ["email: Invalid format"]
        """
        ...
```

## 5. Versioning

Plan for interface changes.

### Strategy 1: Semantic Versioning

```typescript
// v1/interfaces/payment.interface.ts
export interface PaymentProcessorV1 {
  processPayment(amount: number): Promise<PaymentResult>;
}

// v2/interfaces/payment.interface.ts
export interface PaymentProcessorV2 {
  processPayment(amount: number, currency: string): Promise<PaymentResult>;
  //                                ^^^^^^^^ New required parameter
}
```

### Strategy 2: Optional Parameters

```typescript
// Backward compatible - add optional parameters
export interface PaymentProcessor {
  processPayment(
    amount: number,
    currency?: string  // Optional - defaults internally
  ): Promise<PaymentResult>;
}
```

### Strategy 3: Extension

```typescript
// Original interface
export interface PaymentProcessor {
  processPayment(amount: number): Promise<PaymentResult>;
}

// Extended interface (implements original)
export interface ExtendedPaymentProcessor extends PaymentProcessor {
  processPaymentWithCurrency(
    amount: number,
    currency: string
  ): Promise<PaymentResult>;
}
```

## 6. Separation of Concerns

One interface per responsibility.

### ❌ Bad - Mixed concerns

```typescript
interface UserManager {
  // User CRUD
  createUser(data: UserData): User;
  getUser(id: string): User;

  // Authentication
  login(email: string, password: string): Token;
  logout(token: string): void;

  // Email
  sendWelcomeEmail(user: User): void;
  sendResetEmail(user: User): void;

  // Billing
  createSubscription(userId: string, plan: Plan): Subscription;
  cancelSubscription(userId: string): void;
}
```

### ✅ Good - Separated

```typescript
// One interface per responsibility
interface UserRepository {
  createUser(data: UserData): User;
  getUser(id: string): User;
}

interface AuthenticationService {
  login(email: string, password: string): Token;
  logout(token: string): void;
}

interface EmailService {
  sendWelcomeEmail(user: User): void;
  sendResetEmail(user: User): void;
}

interface BillingService {
  createSubscription(userId: string, plan: Plan): Subscription;
  cancelSubscription(userId: string): void;
}
```

## 7. Dependency Direction

Interfaces should depend on abstractions, not implementations.

### ❌ Bad - Depends on implementation

```typescript
// Tightly coupled to specific database
interface UserRepository {
  findUserByMongoId(id: ObjectId): Promise<MongoDocument>;
  saveToMongoDB(user: MongoDocument): Promise<void>;
}
```

### ✅ Good - Abstract

```typescript
// Database-agnostic
interface UserRepository {
  findById(id: string): Promise<User>;
  save(user: User): Promise<void>;
}

// Can implement with MongoDB, PostgreSQL, or any database
class MongoUserRepository implements UserRepository {
  async findById(id: string): Promise<User> {
    const doc = await this.collection.findOne({ _id: new ObjectId(id) });
    return this.mapToUser(doc);
  }

  async save(user: User): Promise<void> {
    const doc = this.mapToDocument(user);
    await this.collection.updateOne({ _id: doc._id }, { $set: doc });
  }
}
```

## Quick Reference

| Principle | Do | Don't |
|-----------|-----|-------|
| **Minimal** | Only needed methods | Kitchen sink interface |
| **Type-safe** | Explicit types | `any`, `object`, `dict` |
| **Errors** | Document all errors | Surprise exceptions |
| **Docs** | Examples + usage | Just signatures |
| **Versioning** | Plan for changes | Break existing code |
| **SRP** | One responsibility | Mixed concerns |
| **DIP** | Depend on abstractions | Depend on implementations |
