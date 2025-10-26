# TypeScript Error Handling Implementation

Complete error handling implementation for TypeScript/Node.js applications.

## Custom Exception Hierarchy

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

## Global Error Handler (Express)

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

## Dependencies

```json
{
  "dependencies": {
    "winston": "^3.11.0",
    "@sentry/node": "^7.0.0",
    "@sentry/profiling-node": "^1.0.0"
  },
  "devDependencies": {
    "@types/express": "^4.17.0"
  }
}
```
