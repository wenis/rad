---
name: error-handling-standardizer
description: Implements consistent error handling patterns, custom exception hierarchies, structured logging, error tracking integration (Sentry), and user-friendly error messages across the application.
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

## Custom Exception Hierarchy

### TypeScript

```typescript
// errors/base.ts
export class AppError extends Error {
  constructor(
    message: string,
    public code: string,
    public statusCode: number = 500,
    public isOperational: boolean = true
  ) {
    super(message);
    this.name = this.constructor.name;
    Error.captureStackTrace(this, this.constructor);
  }
}

export class ValidationError extends AppError {
  constructor(message: string, public fields?: Record<string, string>) {
    super(message, 'VALIDATION_ERROR', 400);
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

export class UnauthorizedError extends AppError {
  constructor(message: string = 'Unauthorized') {
    super(message, 'UNAUTHORIZED', 401);
  }
}

export class ForbiddenError extends AppError {
  constructor(message: string = 'Forbidden') {
    super(message, 'FORBIDDEN', 403);
  }
}

export class ConflictError extends AppError {
  constructor(message: string) {
    super(message, 'CONFLICT', 409);
  }
}

export class RateLimitError extends AppError {
  constructor(public retryAfter: number) {
    super('Too many requests', 'RATE_LIMIT_EXCEEDED', 429);
  }
}

export class ExternalServiceError extends AppError {
  constructor(service: string, public originalError?: Error) {
    super(`External service error: ${service}`, 'EXTERNAL_SERVICE_ERROR', 502);
    this.isOperational = false; // Not caused by user
  }
}

export class DatabaseError extends AppError {
  constructor(message: string, public originalError?: Error) {
    super(message, 'DATABASE_ERROR', 500);
    this.isOperational = false;
  }
}
```

### Python

```python
# errors/base.py
class AppError(Exception):
    """Base application error"""
    def __init__(
        self,
        message: str,
        code: str,
        status_code: int = 500,
        is_operational: bool = True
    ):
        super().__init__(message)
        self.message = message
        self.code = code
        self.status_code = status_code
        self.is_operational = is_operational


class ValidationError(AppError):
    """Input validation error"""
    def __init__(self, message: str, fields: dict[str, str] | None = None):
        super().__init__(message, 'VALIDATION_ERROR', 400)
        self.fields = fields or {}


class NotFoundError(AppError):
    """Resource not found"""
    def __init__(self, resource: str, id: str | None = None):
        message = f"{resource} with id {id} not found" if id else f"{resource} not found"
        super().__init__(message, 'NOT_FOUND', 404)


class UnauthorizedError(AppError):
    """Not authenticated"""
    def __init__(self, message: str = 'Unauthorized'):
        super().__init__(message, 'UNAUTHORIZED', 401)


class ForbiddenError(AppError):
    """Authenticated but no permission"""
    def __init__(self, message: str = 'Forbidden'):
        super().__init__(message, 'FORBIDDEN', 403)


class ConflictError(AppError):
    """Resource conflict (e.g., duplicate)"""
    def __init__(self, message: str):
        super().__init__(message, 'CONFLICT', 409)


class RateLimitError(AppError):
    """Rate limit exceeded"""
    def __init__(self, retry_after: int):
        super().__init__('Too many requests', 'RATE_LIMIT_EXCEEDED', 429)
        self.retry_after = retry_after


class ExternalServiceError(AppError):
    """External service failure"""
    def __init__(self, service: str, original_error: Exception | None = None):
        super().__init__(
            f"External service error: {service}",
            'EXTERNAL_SERVICE_ERROR',
            502,
            is_operational=False
        )
        self.original_error = original_error


class DatabaseError(AppError):
    """Database operation failed"""
    def __init__(self, message: str, original_error: Exception | None = None):
        super().__init__(message, 'DATABASE_ERROR', 500, is_operational=False)
        self.original_error = original_error
```

## Global Error Handler

### Express (Node.js)

```typescript
// middleware/errorHandler.ts
import { Request, Response, NextFunction } from 'express';
import { AppError } from '../errors/base';
import { logger } from '../logger';
import * as Sentry from '@sentry/node';

export function errorHandler(
  err: Error,
  req: Request,
  res: Response,
  next: NextFunction
) {
  // Log error
  if (err instanceof AppError && err.isOperational) {
    logger.warn({
      error: err.message,
      code: err.code,
      statusCode: err.statusCode,
      path: req.path,
      method: req.method,
    });
  } else {
    // Unexpected error - log with full stack
    logger.error({
      error: err.message,
      stack: err.stack,
      path: req.path,
      method: req.method,
    });

    // Report to Sentry
    Sentry.captureException(err);
  }

  // Send response
  if (err instanceof AppError) {
    return res.status(err.statusCode).json({
      error: {
        code: err.code,
        message: err.message,
        ...(err instanceof ValidationError && err.fields ? { details: err.fields } : {}),
        ...(err instanceof RateLimitError ? { retryAfter: err.retryAfter } : {}),
      },
    });
  }

  // Unknown error - don't leak details
  res.status(500).json({
    error: {
      code: 'INTERNAL_SERVER_ERROR',
      message: process.env.NODE_ENV === 'production'
        ? 'An unexpected error occurred'
        : err.message,
    },
  });
}

// Async error wrapper
export function asyncHandler(fn: Function) {
  return (req: Request, res: Response, next: NextFunction) => {
    Promise.resolve(fn(req, res, next)).catch(next);
  };
}
```

**Usage:**
```typescript
// app.ts
import express from 'express';
import { errorHandler } from './middleware/errorHandler';
import { NotFoundError } from './errors/base';

const app = express();

// Routes
app.use('/api', apiRoutes);

// 404 handler
app.use((req, res, next) => {
  next(new NotFoundError('Route'));
});

// Error handler (must be last)
app.use(errorHandler);

export default app;
```

### FastAPI (Python)

```python
# middleware/error_handler.py
from fastapi import FastAPI, Request, status
from fastapi.responses import JSONResponse
from fastapi.exceptions import RequestValidationError
import logging
import sentry_sdk
from errors.base import AppError

logger = logging.getLogger(__name__)


def setup_error_handlers(app: FastAPI):
    @app.exception_handler(AppError)
    async def app_error_handler(request: Request, exc: AppError):
        # Log operational errors as warnings
        if exc.is_operational:
            logger.warning(
                f"{exc.code}: {exc.message}",
                extra={
                    "code": exc.code,
                    "path": request.url.path,
                    "method": request.method,
                }
            )
        else:
            # Unexpected errors - log with full context
            logger.error(
                f"{exc.code}: {exc.message}",
                exc_info=True,
                extra={
                    "code": exc.code,
                    "path": request.url.path,
                    "method": request.method,
                }
            )
            sentry_sdk.capture_exception(exc)

        response = {
            "error": {
                "code": exc.code,
                "message": exc.message,
            }
        }

        # Add extra fields for specific errors
        if hasattr(exc, 'fields') and exc.fields:
            response["error"]["details"] = exc.fields

        if hasattr(exc, 'retry_after'):
            response["error"]["retryAfter"] = exc.retry_after

        return JSONResponse(
            status_code=exc.status_code,
            content=response
        )

    @app.exception_handler(RequestValidationError)
    async def validation_error_handler(request: Request, exc: RequestValidationError):
        errors = {}
        for error in exc.errors():
            field = '.'.join(str(loc) for loc in error['loc'][1:])
            errors[field] = error['msg']

        return JSONResponse(
            status_code=status.HTTP_422_UNPROCESSABLE_ENTITY,
            content={
                "error": {
                    "code": "VALIDATION_ERROR",
                    "message": "Invalid input",
                    "details": errors,
                }
            }
        )

    @app.exception_handler(Exception)
    async def general_exception_handler(request: Request, exc: Exception):
        # Log unexpected errors
        logger.error(
            f"Unexpected error: {str(exc)}",
            exc_info=True,
            extra={
                "path": request.url.path,
                "method": request.method,
            }
        )

        sentry_sdk.capture_exception(exc)

        return JSONResponse(
            status_code=status.HTTP_500_INTERNAL_SERVER_ERROR,
            content={
                "error": {
                    "code": "INTERNAL_SERVER_ERROR",
                    "message": "An unexpected error occurred",
                }
            }
        )
```

**Usage:**
```python
# main.py
from fastapi import FastAPI
from middleware.error_handler import setup_error_handlers

app = FastAPI()
setup_error_handlers(app)
```

## Error Response Format

**Standard format:**
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

**With request ID:**
```json
{
  "error": {
    "code": "DATABASE_ERROR",
    "message": "Failed to process request",
    "requestId": "req_abc123xyz"
  }
}
```

**With retry info:**
```json
{
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "Too many requests",
    "retryAfter": 60
  }
}
```

## Retry Logic

```typescript
// utils/retry.ts
export async function retry<T>(
  fn: () => Promise<T>,
  options: {
    maxAttempts?: number;
    delay?: number;
    backoff?: 'exponential' | 'linear';
    shouldRetry?: (error: Error) => boolean;
  } = {}
): Promise<T> {
  const {
    maxAttempts = 3,
    delay = 1000,
    backoff = 'exponential',
    shouldRetry = () => true,
  } = options;

  let lastError: Error;

  for (let attempt = 1; attempt <= maxAttempts; attempt++) {
    try {
      return await fn();
    } catch (error) {
      lastError = error as Error;

      if (attempt === maxAttempts || !shouldRetry(lastError)) {
        throw lastError;
      }

      const waitTime = backoff === 'exponential'
        ? delay * Math.pow(2, attempt - 1)
        : delay * attempt;

      await new Promise(resolve => setTimeout(resolve, waitTime));
    }
  }

  throw lastError!;
}

// Usage
const user = await retry(
  () => apiClient.getUser(id),
  {
    maxAttempts: 3,
    delay: 1000,
    backoff: 'exponential',
    shouldRetry: (error) => {
      // Retry on network errors, not on 404
      return error instanceof ExternalServiceError;
    },
  }
);
```

## Circuit Breaker

```typescript
// utils/circuitBreaker.ts
type CircuitState = 'CLOSED' | 'OPEN' | 'HALF_OPEN';

export class CircuitBreaker {
  private state: CircuitState = 'CLOSED';
  private failures = 0;
  private nextAttempt = Date.now();

  constructor(
    private threshold: number = 5,
    private timeout: number = 60000
  ) {}

  async execute<T>(fn: () => Promise<T>): Promise<T> {
    if (this.state === 'OPEN') {
      if (Date.now() < this.nextAttempt) {
        throw new Error('Circuit breaker is OPEN');
      }
      this.state = 'HALF_OPEN';
    }

    try {
      const result = await fn();
      this.onSuccess();
      return result;
    } catch (error) {
      this.onFailure();
      throw error;
    }
  }

  private onSuccess() {
    this.failures = 0;
    this.state = 'CLOSED';
  }

  private onFailure() {
    this.failures++;
    if (this.failures >= this.threshold) {
      this.state = 'OPEN';
      this.nextAttempt = Date.now() + this.timeout;
    }
  }
}

// Usage
const breaker = new CircuitBreaker(5, 60000);

try {
  const data = await breaker.execute(() => externalAPI.getData());
} catch (error) {
  // Circuit is open or request failed
}
```

## Error Logging

```typescript
// logger.ts
import winston from 'winston';

export const logger = winston.createLogger({
  level: process.env.LOG_LEVEL || 'info',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.errors({ stack: true }),
    winston.format.json()
  ),
  transports: [
    new winston.transports.File({ filename: 'error.log', level: 'error' }),
    new winston.transports.File({ filename: 'combined.log' }),
    ...(process.env.NODE_ENV !== 'production'
      ? [new winston.transports.Console({ format: winston.format.simple() })]
      : []),
  ],
});

// Usage
logger.error('Database connection failed', {
  error: err.message,
  stack: err.stack,
  context: { userId, operation: 'createUser' },
});
```

## Sentry Integration

```typescript
// sentry.ts
import * as Sentry from '@sentry/node';
import { ProfilingIntegration } from '@sentry/profiling-node';

export function setupSentry() {
  Sentry.init({
    dsn: process.env.SENTRY_DSN,
    environment: process.env.NODE_ENV,
    tracesSampleRate: 0.1,
    profilesSampleRate: 0.1,
    integrations: [
      new Sentry.Integrations.Http({ tracing: true }),
      new ProfilingIntegration(),
    ],
    beforeSend(event, hint) {
      // Don't send operational errors to Sentry
      const error = hint.originalException;
      if (error instanceof AppError && error.isOperational) {
        return null;
      }
      return event;
    },
  });
}

// Capture exception with context
Sentry.captureException(error, {
  tags: { operation: 'payment' },
  extra: { userId, amount },
  user: { id: userId, email: userEmail },
});
```

## User-Friendly Error Messages

```typescript
// utils/errorMessages.ts
export const ERROR_MESSAGES: Record<string, string> = {
  VALIDATION_ERROR: 'Please check your input and try again',
  NOT_FOUND: 'The requested resource could not be found',
  UNAUTHORIZED: 'Please log in to continue',
  FORBIDDEN: 'You do not have permission to perform this action',
  RATE_LIMIT_EXCEEDED: 'Too many requests. Please try again later',
  DATABASE_ERROR: 'We are experiencing technical difficulties. Please try again',
  EXTERNAL_SERVICE_ERROR: 'A service is temporarily unavailable. Please try again',
};

export function getUserMessage(error: AppError): string {
  return ERROR_MESSAGES[error.code] || 'An unexpected error occurred';
}
```

## Testing Error Handling

```typescript
// errors/base.test.ts
import { NotFoundError, ValidationError } from './base';

describe('Custom Errors', () => {
  it('creates NotFoundError correctly', () => {
    const error = new NotFoundError('User', '123');

    expect(error.message).toBe('User with id 123 not found');
    expect(error.code).toBe('NOT_FOUND');
    expect(error.statusCode).toBe(404);
    expect(error.isOperational).toBe(true);
  });

  it('creates ValidationError with fields', () => {
    const error = new ValidationError('Invalid input', {
      email: 'Invalid email',
      age: 'Must be 18+',
    });

    expect(error.statusCode).toBe(400);
    expect(error.fields).toEqual({
      email: 'Invalid email',
      age: 'Must be 18+',
    });
  });
});
```

## Best Practices

✅ **DO:**
- Use specific error types
- Include context in errors
- Log errors appropriately
- Use user-friendly messages
- Integrate error tracking (Sentry)
- Test error scenarios
- Document error codes
- Implement retry logic for transient failures

❌ **DON'T:**
- Expose stack traces to users
- Ignore errors
- Use generic error types everywhere
- Log sensitive data (passwords, tokens)
- Swallow errors silently
- Return different formats for errors
- Skip error testing

## Instructions

1. **Create error hierarchy**: Base error + specific types
2. **Implement global handler**: Catch all errors, log, respond
3. **Integrate Sentry**: Report unexpected errors
4. **Add structured logging**: Include context
5. **User-friendly messages**: Don't expose internals
6. **Test error cases**: Unit tests for error scenarios
7. **Document error codes**: API docs with error responses

## Constraints

- Must not expose sensitive information
- Must log errors appropriately
- Must use consistent error format
- Must track unexpected errors (Sentry)
- Must be user-friendly
- Must include request IDs for debugging
