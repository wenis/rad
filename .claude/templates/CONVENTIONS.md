# Conventions

> **Note**: These are sensible defaults. Modify or remove sections as needed for your project.

## Git Commits

We use [Conventional Commits](https://www.conventionalcommits.org/) for clear, semantic commit history.

### Format
```
<type>(<scope>): <subject>

<body>

<footer>
```

### Types
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation only
- `style`: Formatting, missing semicolons, etc (no code change)
- `refactor`: Code change that neither fixes a bug nor adds a feature
- `perf`: Performance improvement
- `test`: Adding or updating tests
- `chore`: Maintenance tasks, dependency updates

### Examples
```
feat: add user authentication
fix: resolve database connection timeout
docs: update API documentation
refactor: simplify payment processing logic
test: add integration tests for user API
chore: upgrade dependencies
```

### Rules
- Use imperative mood ("add" not "added" or "adds")
- Don't capitalize first letter
- No period at the end
- Keep subject line under 72 characters
- Use body to explain "what" and "why", not "how"

---

## API Standards

### REST Conventions
- **Endpoints**: Use nouns, plural: `/api/v1/users`, `/api/v1/posts`
- **Versioning**: Include version in URL: `/api/v1/`
- **HTTP Methods**:
  - `GET` - Retrieve resource(s)
  - `POST` - Create resource
  - `PUT` - Replace resource
  - `PATCH` - Partial update
  - `DELETE` - Remove resource

### Status Codes
- `200` OK - Successful GET, PUT, PATCH, DELETE
- `201` Created - Successful POST
- `204` No Content - Successful DELETE with no response body
- `400` Bad Request - Invalid request (validation error)
- `401` Unauthorized - Not authenticated
- `403` Forbidden - Authenticated but no permission
- `404` Not Found - Resource doesn't exist
- `409` Conflict - Duplicate resource, version conflict
- `422` Unprocessable Entity - Semantic validation error
- `429` Too Many Requests - Rate limit exceeded
- `500` Internal Server Error - Unexpected server error
- `502` Bad Gateway - External service error
- `503` Service Unavailable - Temporary outage

### Response Format

**Success (resource):**
```json
{
  "id": "123",
  "email": "user@example.com",
  "name": "John Doe",
  "createdAt": "2025-01-24T10:30:00Z"
}
```

**Success (list):**
```json
{
  "data": [
    { "id": "1", "name": "Item 1" },
    { "id": "2", "name": "Item 2" }
  ],
  "meta": {
    "total": 100,
    "page": 1,
    "perPage": 20
  }
}
```

**Error:**
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

**Error Codes:**
- `VALIDATION_ERROR` - Input validation failed
- `NOT_FOUND` - Resource not found
- `UNAUTHORIZED` - Not authenticated
- `FORBIDDEN` - No permission
- `CONFLICT` - Resource conflict (duplicate, etc.)
- `RATE_LIMIT_EXCEEDED` - Too many requests
- `INTERNAL_ERROR` - Unexpected server error
- `EXTERNAL_SERVICE_ERROR` - Third-party service failed

---

## Code Quality

### Automated Formatting
**Don't argue about style - automate it.**

- **TypeScript/JavaScript**: [Prettier](https://prettier.io/)
  ```bash
  npm install -D prettier
  npx prettier --write .
  ```

- **Python**: [Black](https://black.readthedocs.io/)
  ```bash
  pip install black
  black .
  ```

- **Go**: `gofmt` (built-in)
  ```bash
  gofmt -w .
  ```

### Linting
- **TypeScript/JavaScript**: [ESLint](https://eslint.org/)
- **Python**: [Ruff](https://docs.astral.sh/ruff/) (fast, modern)
- **Go**: `golangci-lint`

### Type Safety
- **TypeScript**: Enable `strict` mode in `tsconfig.json`
- **Python**: Use type hints + [mypy](https://mypy-lang.org/)
- **Go**: Built-in static typing

### Naming Conventions

**Files and Folders:**
- Use `kebab-case`: `user-service.ts`, `api-client.py`
- Components (React/Vue): `PascalCase`: `UserProfile.tsx`

**Code:**
- Functions/variables: `camelCase` (JS/TS), `snake_case` (Python/Go)
- Classes/Components: `PascalCase`
- Constants: `UPPER_SNAKE_CASE`
- Private members: `_leadingUnderscore` (Python), `#private` (TS)

---

## Testing

### Requirements
- **Minimum coverage**: 80% for new code
- **All PRs**: Must include tests for new features/fixes
- **Critical paths**: Must have integration tests (auth, payments, data integrity)

### Test Types
- **Unit tests**: Test individual functions (fast, isolated)
- **Integration tests**: Test components together (API endpoints, database)
- **E2E tests**: Test user flows through UI (critical paths only)

### Testing Tools
- **JavaScript/TypeScript**: Jest, Vitest, or Playwright
- **Python**: pytest
- **Go**: built-in `testing` package

---

## Pull Requests

### Requirements
- [ ] All tests passing
- [ ] Code coverage ≥ 80%
- [ ] Linting and formatting passing
- [ ] At least one approval
- [ ] No merge conflicts

### PR Title
Use conventional commit format:
```
feat: add user authentication
fix: resolve database connection issue
```

### PR Description Template
```markdown
## Description
Brief summary of changes and motivation.

## Type of Change
- [ ] Bug fix
- [ ] New feature
- [ ] Breaking change
- [ ] Documentation update

## Testing
- [ ] Unit tests added/updated
- [ ] Integration tests added/updated
- [ ] Tested manually

## Checklist
- [ ] Code follows style guidelines
- [ ] Self-review completed
- [ ] Documentation updated
- [ ] No new warnings
```

### Merge Strategy
- **Squash and merge** to `main` (clean history)
- Delete branch after merge
- Deploy to staging automatically
- Deploy to production manually (with approval)

---

## Security

### Authentication
- All API endpoints require authentication (except `/health`, `/docs`, `/metrics`)
- Use JWT tokens with proper expiration (15 min access, 7 day refresh)
- Store tokens securely (httpOnly cookies or secure storage)

### Input Validation
- Validate all user inputs (use Zod, Pydantic, etc.)
- Sanitize HTML/SQL to prevent injection attacks
- Use parameterized queries (never string concatenation)

### Secrets Management
- **Never commit secrets** to git
- Use environment variables
- Use secret managers in production (AWS Secrets Manager, etc.)
- Rotate secrets regularly

### Rate Limiting
- **Authenticated users**: 100 requests/minute
- **Anonymous users**: 20 requests/minute
- **Login endpoint**: 5 attempts per 15 minutes

---

## Performance

### Targets
- **API Response Time**: P95 < 500ms
- **Page Load Time**: LCP < 2.5s
- **Database Queries**: < 100ms for simple queries
- **Lighthouse Score**: ≥ 90 (Performance, Accessibility, Best Practices, SEO)

### Best Practices
- Use database indexes for frequently queried fields
- Implement caching (Redis) for expensive operations
- Use pagination for large datasets (default: 20 items)
- Optimize images (WebP, lazy loading)
- Use CDN for static assets

---

## Documentation

### Code Comments
- **Don't**: Document *what* code does (code should be self-explanatory)
- **Do**: Document *why* decisions were made (complex logic, workarounds)

### Function Documentation
**TypeScript:**
```typescript
/**
 * Calculates user's credit score based on payment history
 * @param userId - The user's unique identifier
 * @param months - Number of months to analyze (default: 12)
 * @returns Credit score between 300-850
 */
function calculateCreditScore(userId: string, months: number = 12): number {
  // Implementation
}
```

**Python:**
```python
def calculate_credit_score(user_id: str, months: int = 12) -> int:
    """Calculate user's credit score based on payment history.

    Args:
        user_id: The user's unique identifier
        months: Number of months to analyze (default: 12)

    Returns:
        Credit score between 300-850
    """
    # Implementation
```

### API Documentation
- Use OpenAPI/Swagger for REST APIs
- Use GraphQL schema documentation for GraphQL
- Keep documentation in sync with code (use code generation)

---

## Deployment

### Environments
- **Development**: Local machine, Docker Compose
- **Staging**: Mirrors production, automatic deployment from `main`
- **Production**: Manual deployment (requires approval)

### CI/CD Pipeline
1. **Test**: Run all tests, check coverage
2. **Lint**: Check code style
3. **Build**: Create artifacts (Docker images, etc.)
4. **Deploy Staging**: Automatic on merge to `main`
5. **Deploy Production**: Manual approval required

### Health Checks
Every service must expose:
- `GET /health` - Returns 200 if healthy
- `GET /metrics` - Prometheus metrics (optional)

---

## Monitoring

### Logging
- **Structured logging**: Use JSON format
- **Log Levels**: ERROR (alerts), WARN (investigate), INFO (important events), DEBUG (verbose)
- **Include**: Request ID, user ID, timestamp, context

### Error Tracking
- Use Sentry or similar for error tracking
- Alert on error rate > 5%
- Alert on critical errors immediately

### Metrics
- Track: Request rate, error rate, response time (P50, P95, P99)
- Alert on: High error rate, slow response time, service down

---

## Support

For questions or changes to these conventions:
- Open an issue for discussion
- Update this document via PR
- Announce changes in #engineering
