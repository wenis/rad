# API Contract Example

## Build Plan
Module C (email service), Module E (reset API) depends on C

## Generated Contract

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

## When to Use API Contracts

Use API contracts (instead of code interfaces) when:

1. **Different languages**: Module C is Python, Module E is Node.js
2. **Microservices**: Modules will be separate services
3. **HTTP/gRPC**: Communication over network protocol
4. **Third-party**: External service integration
5. **Documentation-first**: Need spec before implementation

## Contract Structure

A good API contract includes:

### 1. Metadata
- Module name and file
- Who implements it
- Who depends on it

### 2. Interface Definition
- Function signatures
- HTTP endpoints (if REST)
- gRPC methods (if gRPC)
- GraphQL schema (if GraphQL)

### 3. Parameters
- Name and type
- Required vs optional
- Validation rules
- Examples

### 4. Return Values
- Type definition
- Success structure
- Error structure

### 5. Error Cases
- All possible errors
- When they occur
- How to handle them

### 6. Examples
- Basic usage
- Common patterns
- Error handling

### 7. Behavior
- Step-by-step process
- Side effects
- Idempotency
- Retry logic

### 8. Performance
- Expected latency
- Timeout values
- Rate limits
- Scaling characteristics

### 9. Testing
- How to test
- Mock implementations
- Test data

## OpenAPI/Swagger Example

For REST APIs, use OpenAPI spec:

```yaml
# interfaces/email-service.openapi.yml
openapi: 3.0.0
info:
  title: Email Service
  version: 1.0.0

paths:
  /send-reset-email:
    post:
      summary: Send password reset email
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              required:
                - recipient
                - reset_token
              properties:
                recipient:
                  type: string
                  format: email
                  example: user@example.com
                reset_token:
                  type: string
                  example: abc123
                expires_in:
                  type: integer
                  default: 3600
                  minimum: 300
                  maximum: 86400
      responses:
        '200':
          description: Email sent successfully
          content:
            application/json:
              schema:
                type: object
                properties:
                  success:
                    type: boolean
                  message_id:
                    type: string
        '400':
          description: Invalid request
          content:
            application/json:
              schema:
                type: object
                properties:
                  error:
                    type: string
```

Module E builder can use this OpenAPI spec to:
- Generate client code
- Know exact request/response format
- Test against mock server
- Validate requests
