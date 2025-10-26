---
name: openapi-spec-generator
description: Generates OpenAPI 3.0/3.1 specifications from existing code (FastAPI, NestJS auto-generate) or creates new specs with schema definitions, request/response examples, authentication schemes (Bearer, OAuth2, API Key), and validation rules. Supports Swagger UI, ReDoc, and Stoplight for documentation. Generates TypeScript/Python API clients and mock servers. Use when documenting REST APIs, creating API-first designs, generating client SDKs, enabling API testing tools (Postman, Insomnia), or setting up contract testing.
allowed-tools: Read, Write, Grep, Edit
---

# OpenAPI Spec Generator

You generate comprehensive OpenAPI 3.0/3.1 specifications that document REST APIs and enable auto-generated clients, servers, and interactive documentation.

## When to use
- Documenting existing REST API
- Creating API-first designs (spec before code)
- Generating interactive API documentation (Swagger UI)
- Auto-generating API clients/SDKs
- Validating API requests/responses
- Contract testing between services
- Enabling API mocking

## OpenAPI 3.0/3.1 Basics

### Core Components

**1. Info** - API metadata (title, version, description)
**2. Servers** - Base URLs for different environments
**3. Paths** - API endpoints and operations
**4. Components** - Reusable schemas, parameters, responses
**5. Security** - Authentication schemes
**6. Tags** - Group operations logically

### Minimal OpenAPI Spec

```yaml
openapi: 3.0.3
info:
  title: My API
  version: 1.0.0
  description: A simple API
servers:
  - url: https://api.example.com/v1
paths:
  /users:
    get:
      summary: List users
      responses:
        '200':
          description: Success
          content:
            application/json:
              schema:
                type: array
                items:
                  $ref: '#/components/schemas/User'
components:
  schemas:
    User:
      type: object
      required:
        - id
        - email
      properties:
        id:
          type: string
          format: uuid
        email:
          type: string
          format: email
        name:
          type: string
```

## Complete Implementation Guides

For framework-specific spec generation and advanced patterns:

**FastAPI (Python)** → `examples/fastapi-openapi.md`
- Auto-generated specs
- Pydantic model integration
- Custom examples
- Security schemes

**Express (Node.js)** → `examples/express-openapi.md`
- swagger-jsdoc
- tsoa for TypeScript
- Decorators and annotations

**NestJS (TypeScript)** → `examples/nestjs-openapi.md`
- @nestjs/swagger
- Auto-generated from decorators
- CLI plugin

**Patterns & Best Practices** → `reference/openapi-patterns.md`
- Pagination
- Filtering/sorting
- Error responses
- Versioning strategies

## Quick Start: FastAPI (Auto-Generated)

FastAPI generates OpenAPI specs automatically:

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI(
    title="My API",
    version="1.0.0",
    description="API description",
)

class User(BaseModel):
    id: str
    email: str
    name: str | None = None

@app.get("/users", response_model=list[User], tags=["users"])
async def list_users():
    """List all users."""
    return []

@app.post("/users", response_model=User, status_code=201, tags=["users"])
async def create_user(user: User):
    """Create a new user."""
    return user

# Spec available at /openapi.json
# Swagger UI at /docs
# ReDoc at /redoc
```

## Quick Start: Manual YAML Spec

### Step 1: Define Info & Servers

```yaml
openapi: 3.0.3
info:
  title: E-commerce API
  version: 1.0.0
  description: API for managing products and orders
  contact:
    name: API Support
    email: support@example.com
  license:
    name: MIT
servers:
  - url: https://api.example.com/v1
    description: Production
  - url: https://staging-api.example.com/v1
    description: Staging
```

### Step 2: Define Schemas (Components)

```yaml
components:
  schemas:
    Product:
      type: object
      required:
        - id
        - name
        - price
      properties:
        id:
          type: string
          format: uuid
          example: "123e4567-e89b-12d3-a456-426614174000"
        name:
          type: string
          minLength: 1
          maxLength: 200
          example: "Laptop"
        price:
          type: number
          format: float
          minimum: 0
          example: 999.99
        description:
          type: string
          example: "High-performance laptop"

    Error:
      type: object
      required:
        - code
        - message
      properties:
        code:
          type: string
        message:
          type: string
        details:
          type: object
```

### Step 3: Define Paths

```yaml
paths:
  /products:
    get:
      summary: List products
      tags:
        - products
      parameters:
        - name: page
          in: query
          schema:
            type: integer
            default: 1
        - name: limit
          in: query
          schema:
            type: integer
            default: 20
            maximum: 100
      responses:
        '200':
          description: Success
          content:
            application/json:
              schema:
                type: object
                properties:
                  data:
                    type: array
                    items:
                      $ref: '#/components/schemas/Product'
                  page:
                    type: integer
                  total:
                    type: integer
        '500':
          description: Server error
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Error'

    post:
      summary: Create product
      tags:
        - products
      security:
        - bearerAuth: []
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/Product'
      responses:
        '201':
          description: Created
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Product'
        '400':
          description: Validation error
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Error'
```

### Step 4: Add Security

```yaml
components:
  securitySchemes:
    bearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT
    apiKey:
      type: apiKey
      in: header
      name: X-API-Key
```

## Common Patterns

### Pagination

```yaml
components:
  parameters:
    PageParam:
      name: page
      in: query
      schema:
        type: integer
        minimum: 1
        default: 1
    LimitParam:
      name: limit
      in: query
      schema:
        type: integer
        minimum: 1
        maximum: 100
        default: 20

  schemas:
    PaginatedResponse:
      type: object
      properties:
        data:
          type: array
          items: {}
        page:
          type: integer
        limit:
          type: integer
        total:
          type: integer
        hasMore:
          type: boolean
```

### Error Responses

```yaml
components:
  responses:
    NotFound:
      description: Resource not found
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/Error'
          example:
            code: "NOT_FOUND"
            message: "Resource not found"

    ValidationError:
      description: Validation failed
      content:
        application/json:
          schema:
            type: object
            properties:
              code:
                type: string
              message:
                type: string
              details:
                type: object
                additionalProperties:
                  type: string
          example:
            code: "VALIDATION_ERROR"
            message: "Invalid input"
            details:
              email: "Invalid email format"
```

### Filtering

```yaml
paths:
  /products:
    get:
      parameters:
        - name: category
          in: query
          schema:
            type: string
            enum: [electronics, clothing, books]
        - name: minPrice
          in: query
          schema:
            type: number
        - name: maxPrice
          in: query
          schema:
            type: number
        - name: sort
          in: query
          schema:
            type: string
            enum: [price_asc, price_desc, name_asc, name_desc]
```

## Tools & Validation

### Validate Spec

```bash
# Swagger Editor (online)
# https://editor.swagger.io

# CLI validation
npm install -g @apidevtools/swagger-cli
swagger-cli validate openapi.yaml

# OpenAPI Generator CLI
npm install -g @openapitools/openapi-generator-cli
openapi-generator-cli validate -i openapi.yaml
```

### Generate Interactive Docs

```bash
# Swagger UI
npx serve-swagger-ui openapi.yaml

# ReDoc
npx redoc-cli serve openapi.yaml

# Docker
docker run -p 8080:8080 -e SWAGGER_JSON=/openapi.yaml -v $(pwd):/usr/share/nginx/html swaggerapi/swagger-ui
```

### Generate Clients/Servers

```bash
# Generate TypeScript client
openapi-generator-cli generate -i openapi.yaml -g typescript-axios -o ./client

# Generate Python server (FastAPI)
openapi-generator-cli generate -i openapi.yaml -g python-fastapi -o ./server

# Generate Go client
openapi-generator-cli generate -i openapi.yaml -g go -o ./client
```

## Examples & Documentation

### Request/Response Examples

```yaml
paths:
  /users:
    post:
      requestBody:
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/User'
            examples:
              minimal:
                value:
                  email: "user@example.com"
              complete:
                value:
                  email: "user@example.com"
                  name: "John Doe"
                  age: 30
      responses:
        '201':
          content:
            application/json:
              examples:
                success:
                  value:
                    id: "123e4567-e89b-12d3-a456-426614174000"
                    email: "user@example.com"
                    name: "John Doe"
```

### Rich Descriptions

Use markdown in descriptions for filtering, sorting, rate limits, and usage examples.

## Best Practices

**DO:** Use OpenAPI 3.0+, define reusable components ($ref), add examples, validate spec, version API, document all errors and security

**DON'T:** Duplicate schemas, skip examples, ignore validation, use vague descriptions, omit required fields

## Instructions

1. **Identify approach**: Auto-generate (FastAPI, NestJS), code annotations (swagger-jsdoc), or manual YAML
2. **Define info & servers**: API title, version, description, server URLs
3. **Create schemas**: Data models, common responses, reusable parameters in components
4. **Document paths**: All endpoints with request/response schemas, examples, security
5. **Add security schemes**: Bearer token, API key, OAuth2 - apply to operations
6. **Validate spec**: swagger-cli validate, test in Swagger Editor
7. **Generate documentation**: Swagger UI or ReDoc, deploy to docs site
8. **Optional: Generate code**: API clients, server stubs, mock servers

## Constraints

- Must use OpenAPI 3.0+ (not Swagger 2.0)
- Must validate spec before committing
- Must include examples for all operations
- Must document all response codes (including errors)
- Must use components/$ref to avoid duplication
- Should include descriptions for all operations
- Should specify security requirements per operation
- Should version the API in the URL
- Must keep spec in sync with code
- Should auto-generate when possible (FastAPI, NestJS)
