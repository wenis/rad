---
name: error-handling-standardizer
description: Implements standardized error handling with custom exception hierarchies, structured error responses (REST/GraphQL), user-friendly error messages, retry logic, circuit breakers, and error tracking integration (Sentry/Rollbar). Creates typed error classes for TypeScript/Python, implements graceful degradation, and sets up error monitoring dashboards. Use when building robust APIs, standardizing error responses, implementing retry patterns, adding circuit breakers, integrating error tracking, or improving error messages for better debugging.
allowed-tools: Read, Write, Edit
---

# Error Handling Standardizer

You implement consistent, production-ready error handling that makes debugging easy and provides great user experience.

## When to use
- Building new application (establish patterns early)
- Refactoring inconsistent error handling
- Integrating error tracking (Sentry, Rollbar)
- Improving error messages for users
- Adding structured logging
- Preparing for production

## Error Handling Principles

### 1. Fail Fast
Detect errors as early as possible, close to the source.

### 2. Be Specific
Use specific error types, not generic exceptions.

### 3. Provide Context
Include relevant information for debugging.

### 4. Log Appropriately
Critical errors logged differently than warnings.

### 5. User-Friendly Messages
Never expose stack traces to users.

### 6. Recover Gracefully
Handle errors without crashing the entire application.

## Quick Start: Basic Error Class

Here's a minimal custom error implementation:

```typescript
// TypeScript
export class AppError extends Error {
  constructor(
    message: string,
    public code: string,
    public statusCode: number = 500
  ) {
    super(message);
    this.name = this.constructor.name;
  }
}

export class NotFoundError extends AppError {
  constructor(resource: string, id?: string) {
    const message = id
      ? `${resource} with id ${id} not found`
      : `${resource} not found`;
    super(message, 'NOT_FOUND', 404);
  }
}

// Usage
throw new NotFoundError('User', userId);
```

```python
# Python
class AppError(Exception):
    def __init__(self, message: str, code: str, status_code: int = 500):
        super().__init__(message)
        self.message = message
        self.code = code
        self.status_code = status_code

class NotFoundError(AppError):
    def __init__(self, resource: str, id: str | None = None):
        message = f"{resource} with id {id} not found" if id else f"{resource} not found"
        super().__init__(message, 'NOT_FOUND', 404)

# Usage
raise NotFoundError('User', user_id)
```

## Complete Language-Specific Implementations

For full production-ready error handling systems:

**TypeScript/Node.js** → `examples/typescript-implementation.md`
- Complete custom exception hierarchy
- Express global error handler
- Async error wrapper
- Winston logging setup
- Sentry integration
- User-friendly error messages
- Testing examples

**Python/FastAPI** → `examples/python-implementation.md`
- Complete custom exception hierarchy
- FastAPI global error handlers
- Structured logging configuration
- Sentry integration
- User-friendly error messages
- Testing examples

## Advanced Resilience Patterns

**Retry Logic & Circuit Breaker** → `reference/advanced-patterns.md`
- Retry with exponential backoff
- Circuit breaker pattern
- When to use each pattern
- Both TypeScript and Python implementations

## Standard Error Types

Your custom error hierarchy should include:

### Client Errors (4xx)

**ValidationError (400)**
- Invalid input data
- Include field-level error details

**UnauthorizedError (401)**
- Not authenticated
- Missing or invalid credentials

**ForbiddenError (403)**
- Authenticated but lacks permission
- Role/permission issues

**NotFoundError (404)**
- Resource doesn't exist
- Include resource type and ID

**ConflictError (409)**
- Resource already exists
- Duplicate violations

**RateLimitError (429)**
- Too many requests
- Include retry-after time

### Server Errors (5xx)

**DatabaseError (500)**
- Database operation failed
- Mark as non-operational

**ExternalServiceError (502)**
- Third-party service failure
- Mark as non-operational

## Error Response Format

Use a consistent JSON format for all errors:

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid input data",
    "details": {
      "email": "Invalid email format",
      "age": "Must be at least 18"
    }
  }
}
```

**With request ID for debugging:**
```json
{
  "error": {
    "code": "DATABASE_ERROR",
    "message": "Failed to process request",
    "requestId": "req_abc123xyz"
  }
}
```

**With retry information:**
```json
{
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "Too many requests",
    "retryAfter": 60
  }
}
```

## Global Error Handler Pattern

### Key Responsibilities

1. **Catch all unhandled errors**
   - Last line of defense
   - Prevent application crash

2. **Log appropriately**
   - Operational errors → warning
   - Unexpected errors → error with stack trace

3. **Report to error tracking**
   - Only unexpected errors to Sentry
   - Skip operational/user errors

4. **Return consistent format**
   - Hide internals in production
   - Include debug info in development

5. **Set correct HTTP status**
   - Map error types to status codes

### Express Example

```typescript
app.use((err, req, res, next) => {
  // Log
  if (err.isOperational) {
    logger.warn({ error: err.message, code: err.code });
  } else {
    logger.error({ error: err.message, stack: err.stack });
    Sentry.captureException(err);
  }

  // Respond
  if (err instanceof AppError) {
    return res.status(err.statusCode).json({
      error: { code: err.code, message: err.message }
    });
  }

  // Unknown error
  res.status(500).json({
    error: {
      code: 'INTERNAL_SERVER_ERROR',
      message: process.env.NODE_ENV === 'production'
        ? 'An unexpected error occurred'
        : err.message
    }
  });
});
```

### FastAPI Example

```python
@app.exception_handler(AppError)
async def app_error_handler(request: Request, exc: AppError):
    if exc.is_operational:
        logger.warning(f"{exc.code}: {exc.message}")
    else:
        logger.error(f"{exc.code}: {exc.message}", exc_info=True)
        sentry_sdk.capture_exception(exc)

    return JSONResponse(
        status_code=exc.status_code,
        content={"error": {"code": exc.code, "message": exc.message}}
    )
```

## Error Logging Best Practices

```typescript
// Good: Include context
logger.error('Payment processing failed', {
  userId: user.id,
  amount: payment.amount,
  error: err.message,
  stack: err.stack
});

// Bad: No context
logger.error('Error occurred');
```

**Log levels:**
- `ERROR`: Unexpected errors, requires investigation
- `WARN`: Operational errors (validation, not found)
- `INFO`: Normal operations
- `DEBUG`: Detailed debugging information

## Sentry Integration Pattern

```typescript
// Initialize
Sentry.init({
  dsn: process.env.SENTRY_DSN,
  environment: process.env.NODE_ENV,
  beforeSend(event, hint) {
    // Don't send operational errors
    const error = hint.originalException;
    if (error instanceof AppError && error.isOperational) {
      return null;
    }
    return event;
  },
});

// Capture with context
Sentry.captureException(error, {
  tags: { operation: 'payment' },
  extra: { userId, amount },
  user: { id: userId, email: userEmail },
});
```

## User-Friendly Error Messages

Never expose:
- Stack traces
- Database errors
- Internal paths
- API keys or secrets

Instead, provide:
- Clear, actionable messages
- What went wrong
- How to fix it
- Support contact (if applicable)

```typescript
const ERROR_MESSAGES = {
  VALIDATION_ERROR: 'Please check your input and try again',
  NOT_FOUND: 'The requested resource could not be found',
  UNAUTHORIZED: 'Please log in to continue',
  RATE_LIMIT_EXCEEDED: 'Too many requests. Please try again later',
  DATABASE_ERROR: 'We are experiencing technical difficulties',
};
```

## Testing Error Handling

```typescript
describe('Error Handling', () => {
  it('throws NotFoundError for missing user', () => {
    const error = new NotFoundError('User', '123');

    expect(error.code).toBe('NOT_FOUND');
    expect(error.statusCode).toBe(404);
    expect(error.message).toBe('User with id 123 not found');
  });

  it('returns correct error response', async () => {
    const response = await request(app)
      .get('/users/invalid')
      .expect(404);

    expect(response.body.error.code).toBe('NOT_FOUND');
  });
});
```

## Best Practices

✅ **DO:**
- Use specific error types for different scenarios
- Include context in error messages (resource, ID, operation)
- Log errors with appropriate levels
- Integrate error tracking (Sentry) for unexpected errors
- Use user-friendly messages in responses
- Test error scenarios
- Document error codes in API documentation
- Implement retry logic for transient failures
- Use circuit breakers for external services

❌ **DON'T:**
- Expose stack traces to users
- Ignore errors silently
- Use generic error types for everything
- Log sensitive data (passwords, tokens, PII)
- Swallow errors without handling
- Return different error formats across endpoints
- Skip testing error cases
- Send operational errors to Sentry (pollutes data)
- Retry non-idempotent operations blindly

## Instructions

1. **Create error hierarchy**
   - Base `AppError` class
   - Specific error types (NotFound, Validation, Unauthorized, etc.)
   - Include `code`, `statusCode`, `isOperational` fields

2. **Implement global error handler**
   - Catch all errors
   - Log appropriately (warn vs error)
   - Send consistent JSON response
   - Hide internals in production

3. **Integrate error tracking**
   - Set up Sentry
   - Filter operational errors
   - Add context to exceptions

4. **Add structured logging**
   - Use JSON format
   - Include request context
   - Log at appropriate levels

5. **Create user-friendly messages**
   - Map error codes to messages
   - Actionable guidance
   - No technical details

6. **Test error scenarios**
   - Unit tests for error classes
   - Integration tests for error responses
   - Test logging and Sentry integration

7. **Document error codes**
   - API documentation
   - Error code reference
   - Example responses

8. **Add resilience patterns** (optional)
   - Retry logic for transient failures
   - Circuit breakers for external services
   - See `reference/advanced-patterns.md`

## Constraints

- Must not expose sensitive information (stack traces, internals)
- Must log errors appropriately (warn vs error)
- Must use consistent error response format across all endpoints
- Must track unexpected errors with Sentry
- Must be user-friendly (actionable messages)
- Must include request IDs for correlation
- Error messages must be internationalization-ready (if applicable)
- Must distinguish operational vs programming errors
- Must not send operational errors to Sentry (reduces noise)
