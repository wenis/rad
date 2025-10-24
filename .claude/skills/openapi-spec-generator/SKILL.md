---
name: openapi-spec-generator
description: Generates OpenAPI/Swagger specifications from code, documenting REST APIs automatically. Use when documenting APIs OR after building API endpoints OR setting up API documentation OR before generating API clients.
allowed-tools: Read, Grep, Write, Bash
---

# OpenAPI Spec Generator

You generate OpenAPI 3.0 specifications from existing code or route definitions, creating comprehensive API documentation automatically.

## When to use
- After building REST API endpoints
- When API documentation is outdated
- Before generating API clients (for consumers)
- Setting up API documentation site (Swagger UI, Redoc)
- Preparing for API versioning
- Onboarding new developers

## What is OpenAPI?

OpenAPI Specification (formerly Swagger) is a standard for describing REST APIs. It includes:
- Available endpoints and operations
- Request/response formats
- Authentication methods
- Rate limits and other metadata

**Benefits:**
- Auto-generated interactive documentation (Swagger UI)
- Client SDK generation (api-client-generator skill)
- API testing tools integration
- Contract testing between services

## Generation Approaches

### 1. From Code Annotations (Best)
Frameworks with built-in OpenAPI support.

### 2. From Route Definitions
Parse routes and infer types from code.

### 3. From Runtime Introspection
Run app and capture requests/responses.

### 4. Manual Creation
Write spec by hand (tedious, use as last resort).

## Framework-Specific Generation

### FastAPI (Python) - Built-in OpenAPI

FastAPI automatically generates OpenAPI specs!

**Code:**
```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel

app = FastAPI(
    title="My API",
    description="API for managing users",
    version="1.0.0"
)

class User(BaseModel):
    id: int
    email: str
    name: str

@app.get("/users/{user_id}", response_model=User, tags=["users"])
async def get_user(user_id: int):
    """Get a user by ID."""
    # Implementation
    return {"id": user_id, "email": "user@example.com", "name": "John"}

@app.post("/users", response_model=User, status_code=201, tags=["users"])
async def create_user(user: User):
    """Create a new user."""
    # Implementation
    return user
```

**Get spec:**
```bash
# OpenAPI JSON available at /openapi.json
curl http://localhost:8000/openapi.json > openapi.json

# Interactive docs at /docs (Swagger UI)
# Alternative docs at /redoc (ReDoc)
```

### Flask (Python) - Using flask-apispec

**Install:**
```bash
pip install flask-apispec marshmallow
```

**Code:**
```python
from flask import Flask
from flask_apispec import FlaskApiSpec, use_kwargs, marshal_with
from marshmallow import Schema, fields

app = Flask(__name__)
app.config['APISPEC_SPEC'] = {
    'title': 'My API',
    'version': '1.0.0',
    'openapi_version': '3.0.0'
}
docs = FlaskApiSpec(app)

class UserSchema(Schema):
    id = fields.Int(required=True)
    email = fields.Str(required=True)
    name = fields.Str(required=True)

@app.route('/users/<int:user_id>', methods=['GET'])
@marshal_with(UserSchema)
def get_user(user_id):
    """Get user by ID"""
    return {'id': user_id, 'email': 'user@example.com', 'name': 'John'}

docs.register(get_user)

# Access spec at /api/swagger.json
```

### Express (Node.js) - Using swagger-jsdoc

**Install:**
```bash
npm install swagger-jsdoc swagger-ui-express
```

**Code:**
```javascript
const express = require('express');
const swaggerJsdoc = require('swagger-jsdoc');
const swaggerUi = require('swagger-ui-express');

const app = express();

const swaggerOptions = {
  definition: {
    openapi: '3.0.0',
    info: {
      title: 'My API',
      version: '1.0.0',
      description: 'API for managing users',
    },
    servers: [{
      url: 'http://localhost:3000',
      description: 'Development server',
    }],
  },
  apis: ['./routes/*.js'],
};

const swaggerSpec = swaggerJsdoc(swaggerOptions);
app.use('/api-docs', swaggerUi.serve, swaggerUi.setup(swaggerSpec));

/**
 * @openapi
 * /users/{userId}:
 *   get:
 *     summary: Get user by ID
 *     tags: [Users]
 *     parameters:
 *       - in: path
 *         name: userId
 *         required: true
 *         schema:
 *           type: integer
 *     responses:
 *       200:
 *         description: User object
 *         content:
 *           application/json:
 *             schema:
 *               type: object
 *               properties:
 *                 id:
 *                   type: integer
 *                 email:
 *                   type: string
 *                 name:
 *                   type: string
 */
app.get('/users/:userId', (req, res) => {
  res.json({ id: req.params.userId, email: 'user@example.com', name: 'John' });
});
```

### NestJS (TypeScript) - Built-in OpenAPI

**Install:**
```bash
npm install @nestjs/swagger
```

**Code:**
```typescript
import { Controller, Get, Post, Body, Param } from '@nestjs/common';
import { ApiTags, ApiOperation, ApiResponse } from '@nestjs/swagger';

class UserDto {
  id: number;
  email: string;
  name: string;
}

@ApiTags('users')
@Controller('users')
export class UsersController {
  @Get(':id')
  @ApiOperation({ summary: 'Get user by ID' })
  @ApiResponse({ status: 200, description: 'User found', type: UserDto })
  @ApiResponse({ status: 404, description: 'User not found' })
  async getUser(@Param('id') id: number): Promise<UserDto> {
    return { id, email: 'user@example.com', name: 'John' };
  }

  @Post()
  @ApiOperation({ summary: 'Create user' })
  @ApiResponse({ status: 201, description: 'User created', type: UserDto })
  async createUser(@Body() user: UserDto): Promise<UserDto> {
    return user;
  }
}
```

**Main file:**
```typescript
import { NestFactory } from '@nestjs/core';
import { SwaggerModule, DocumentBuilder } from '@nestjs/swagger';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  const config = new DocumentBuilder()
    .setTitle('My API')
    .setDescription('API for managing users')
    .setVersion('1.0')
    .build();

  const document = SwaggerModule.createDocument(app, config);
  SwaggerModule.setup('api-docs', app, document);

  await app.listen(3000);
}
```

### Django REST Framework (Python)

**Install:**
```bash
pip install drf-spectacular
```

**Settings:**
```python
INSTALLED_APPS = [
    # ...
    'drf_spectacular',
]

REST_FRAMEWORK = {
    'DEFAULT_SCHEMA_CLASS': 'drf_spectacular.openapi.AutoSchema',
}

SPECTACULAR_SETTINGS = {
    'TITLE': 'My API',
    'DESCRIPTION': 'API for managing users',
    'VERSION': '1.0.0',
}
```

**URLs:**
```python
from drf_spectacular.views import SpectacularAPIView, SpectacularSwaggerView

urlpatterns = [
    path('api/schema/', SpectacularAPIView.as_view(), name='schema'),
    path('api/docs/', SpectacularSwaggerView.as_view(url_name='schema'), name='swagger-ui'),
]
```

**Views:**
```python
from rest_framework import viewsets
from drf_spectacular.utils import extend_schema, OpenApiParameter

class UserViewSet(viewsets.ModelViewSet):
    queryset = User.objects.all()
    serializer_class = UserSerializer

    @extend_schema(
        summary="Get user by ID",
        responses={200: UserSerializer, 404: None},
    )
    def retrieve(self, request, *args, **kwargs):
        return super().retrieve(request, *args, **kwargs)
```

## Manual Spec Creation

When frameworks don't have good support, create spec manually:

**Basic structure:**
```yaml
openapi: 3.0.0
info:
  title: My API
  description: API for managing users
  version: 1.0.0
  contact:
    name: API Support
    email: api@example.com

servers:
  - url: https://api.example.com/v1
    description: Production server
  - url: http://localhost:8000
    description: Development server

tags:
  - name: users
    description: User management operations

paths:
  /users/{userId}:
    get:
      summary: Get user by ID
      description: Retrieve a single user by their unique identifier
      operationId: getUserById
      tags:
        - users
      parameters:
        - name: userId
          in: path
          required: true
          description: Unique user identifier
          schema:
            type: integer
            minimum: 1
      responses:
        '200':
          description: Successful response
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/User'
        '404':
          description: User not found
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Error'
        '500':
          description: Internal server error

  /users:
    post:
      summary: Create user
      description: Create a new user account
      operationId: createUser
      tags:
        - users
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/CreateUserRequest'
      responses:
        '201':
          description: User created successfully
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/User'
        '400':
          description: Invalid input
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Error'

components:
  schemas:
    User:
      type: object
      required:
        - id
        - email
        - name
      properties:
        id:
          type: integer
          example: 1
          description: Unique user identifier
        email:
          type: string
          format: email
          example: user@example.com
          description: User's email address
        name:
          type: string
          example: John Doe
          description: User's full name
        createdAt:
          type: string
          format: date-time
          example: "2025-01-15T10:30:00Z"
          description: Account creation timestamp

    CreateUserRequest:
      type: object
      required:
        - email
        - name
      properties:
        email:
          type: string
          format: email
          example: newuser@example.com
        name:
          type: string
          example: Jane Doe

    Error:
      type: object
      properties:
        code:
          type: string
          example: USER_NOT_FOUND
        message:
          type: string
          example: The requested user was not found

  securitySchemes:
    bearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT

security:
  - bearerAuth: []
```

## Generating from Existing Code

If your framework doesn't have OpenAPI support, generate by analyzing routes:

**Step 1: Extract routes**
```bash
# Express: Find route definitions
grep -rn "app\.\(get\|post\|put\|delete\|patch\)" routes/ > routes.txt

# Flask: Find route decorators
grep -rn "@app.route\|@bp.route" . > routes.txt
```

**Step 2: Analyze route handlers**
```bash
# Find handler functions
for route in $(cat routes.txt); do
  # Extract function signature
  # Identify request/response types
  # Generate OpenAPI path object
done
```

**Step 3: Infer schemas from types**
```python
# Python: Use type hints
def create_user(user: CreateUserRequest) -> User:
    # Types can be converted to OpenAPI schemas
```

```typescript
// TypeScript: Use interfaces
interface User {
  id: number;
  email: string;
  name: string;
}
```

## Setting Up Documentation UI

### Swagger UI

**Docker:**
```yaml
# docker-compose.yml
version: '3.8'
services:
  swagger-ui:
    image: swaggerapi/swagger-ui
    ports:
      - "8080:8080"
    environment:
      SWAGGER_JSON: /app/openapi.json
    volumes:
      - ./openapi.json:/app/openapi.json
```

**Run:**
```bash
docker-compose up swagger-ui
# Access at http://localhost:8080
```

### ReDoc

**Docker:**
```yaml
services:
  redoc:
    image: redocly/redoc
    ports:
      - "8080:80"
    environment:
      SPEC_URL: /openapi.json
    volumes:
      - ./openapi.json:/usr/share/nginx/html/openapi.json
```

## Validation and Testing

**Validate spec:**
```bash
# Using openapi-generator-cli
docker run --rm -v ${PWD}:/local openapitools/openapi-generator-cli validate -i /local/openapi.json

# Using swagger-cli
npx swagger-cli validate openapi.json
```

**Generate sample requests:**
```bash
# Using openapi-examples-validator
npx openapi-examples-validator openapi.json
```

## Instructions

1. **Identify framework:**
   - Detect which framework is used (FastAPI, Express, Flask, etc.)
   - Check if it has built-in OpenAPI support

2. **If built-in support exists:**
   - Add necessary annotations/decorators
   - Configure OpenAPI settings
   - Generate spec via framework

3. **If no built-in support:**
   - Extract routes from code
   - Infer request/response types
   - Build OpenAPI spec manually or with tools

4. **Enrich the spec:**
   - Add descriptions and examples
   - Document authentication
   - Add error responses
   - Include rate limits if applicable

5. **Set up documentation UI:**
   - Deploy Swagger UI or ReDoc
   - Make accessible to developers

6. **Validate:**
   - Run validation tools
   - Test in Swagger UI
   - Ensure examples work

## Output Artifacts

- `openapi.json` or `openapi.yaml` - The spec file
- Documentation UI setup (Docker Compose or similar)
- README with links to docs

## Best Practices

✅ **DO:**
- Include examples for all endpoints
- Document all error responses
- Use semantic versioning for API
- Keep spec in sync with code
- Add authentication/authorization details

❌ **DON'T:**
- Write spec before code (hard to maintain)
- Skip response descriptions
- Forget to version the API
- Leave security undefined
- Ignore deprecated endpoints

## Integration with CI/CD

```yaml
# .github/workflows/api-docs.yml
name: Generate API Docs

on: [push]

jobs:
  docs:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Generate OpenAPI spec
        run: npm run generate:openapi

      - name: Validate spec
        run: npx swagger-cli validate openapi.json

      - name: Deploy docs
        run: |
          docker build -t api-docs .
          docker push registry.example.com/api-docs:latest
```

## Constraints

- Spec must be valid OpenAPI 3.0 format
- Include all endpoints (no undocumented APIs)
- Keep spec in sync with code (automate if possible)
- Document breaking changes with version bumps
