# Python Error Handling Implementation

Complete error handling implementation for Python/FastAPI applications.

## Custom Exception Hierarchy

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

## Global Error Handler (FastAPI)

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
from errors.base import NotFoundError, ValidationError

app = FastAPI()
setup_error_handlers(app)

@app.get("/users/{user_id}")
async def get_user(user_id: str):
    user = await db.get_user(user_id)
    if not user:
        raise NotFoundError("User", user_id)
    return user

@app.post("/users")
async def create_user(email: str, age: int):
    if age < 18:
        raise ValidationError(
            "Invalid user data",
            fields={"age": "Must be at least 18"}
        )
    return await db.create_user(email, age)
```

## Error Logging

```python
# logging_config.py
import logging
import logging.config

LOGGING_CONFIG = {
    "version": 1,
    "disable_existing_loggers": False,
    "formatters": {
        "json": {
            "format": "%(asctime)s %(name)s %(levelname)s %(message)s",
            "class": "pythonjsonlogger.jsonlogger.JsonFormatter",
        },
    },
    "handlers": {
        "console": {
            "class": "logging.StreamHandler",
            "level": "INFO",
            "formatter": "json",
            "stream": "ext://sys.stdout",
        },
        "file": {
            "class": "logging.handlers.RotatingFileHandler",
            "level": "ERROR",
            "formatter": "json",
            "filename": "error.log",
            "maxBytes": 10485760,  # 10MB
            "backupCount": 5,
        },
    },
    "root": {
        "level": "INFO",
        "handlers": ["console", "file"],
    },
}

def setup_logging():
    logging.config.dictConfig(LOGGING_CONFIG)
```

## Sentry Integration

```python
# sentry_config.py
import sentry_sdk
from sentry_sdk.integrations.fastapi import FastApiIntegration
from sentry_sdk.integrations.sqlalchemy import SqlalchemyIntegration
from errors.base import AppError
import os

def setup_sentry():
    sentry_sdk.init(
        dsn=os.environ.get('SENTRY_DSN'),
        environment=os.environ.get('ENV', 'development'),
        traces_sample_rate=0.1,
        profiles_sample_rate=0.1,
        integrations=[
            FastApiIntegration(),
            SqlalchemyIntegration(),
        ],
        before_send=_before_send,
    )

def _before_send(event, hint):
    """Don't send operational errors to Sentry"""
    if 'exc_info' in hint:
        exc_type, exc_value, tb = hint['exc_info']
        if isinstance(exc_value, AppError) and exc_value.is_operational:
            return None
    return event

# Capture exception with context
sentry_sdk.capture_exception(
    error,
    tags={"operation": "payment"},
    extras={"user_id": user_id, "amount": amount},
)
sentry_sdk.set_user({"id": user_id, "email": user_email})
```

## User-Friendly Error Messages

```python
# utils/error_messages.py
ERROR_MESSAGES = {
    'VALIDATION_ERROR': 'Please check your input and try again',
    'NOT_FOUND': 'The requested resource could not be found',
    'UNAUTHORIZED': 'Please log in to continue',
    'FORBIDDEN': 'You do not have permission to perform this action',
    'RATE_LIMIT_EXCEEDED': 'Too many requests. Please try again later',
    'DATABASE_ERROR': 'We are experiencing technical difficulties. Please try again',
    'EXTERNAL_SERVICE_ERROR': 'A service is temporarily unavailable. Please try again',
}

def get_user_message(error: AppError) -> str:
    return ERROR_MESSAGES.get(error.code, 'An unexpected error occurred')
```

## Testing Error Handling

```python
# tests/test_errors.py
import pytest
from errors.base import NotFoundError, ValidationError

def test_not_found_error():
    error = NotFoundError('User', '123')

    assert error.message == 'User with id 123 not found'
    assert error.code == 'NOT_FOUND'
    assert error.status_code == 404
    assert error.is_operational is True

def test_validation_error_with_fields():
    error = ValidationError(
        'Invalid input',
        fields={'email': 'Invalid email', 'age': 'Must be 18+'}
    )

    assert error.status_code == 400
    assert error.fields == {'email': 'Invalid email', 'age': 'Must be 18+'}
```

## Dependencies

```bash
pip install fastapi sentry-sdk python-json-logger
```

Or in `requirements.txt`:
```
fastapi>=0.104.0
sentry-sdk[fastapi]>=1.38.0
python-json-logger>=2.0.7
```
