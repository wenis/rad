---
name: docker-compose-builder
description: Creates Docker and Docker Compose configurations for multi-service applications. Use when containerizing applications OR setting up local development environments OR creating deployment configs OR adding new services to existing compose files.
allowed-tools: Read, Write, Edit, Bash
---

# Docker Compose Builder

You create production-ready Docker and Docker Compose configurations for modern applications.

## When to use
- Setting up local development environment
- Containerizing an application for the first time
- Adding new services (database, cache, queue)
- Creating staging/production deployment configs
- Building multi-service architectures (microservices)

## Core components

### 1. Dockerfile
Application container definition.

### 2. docker-compose.yml
Multi-service orchestration.

### 3. .dockerignore
Files to exclude from build context.

### 4. Environment files
- `.env.example` - Template
- `.env.development` - Dev config
- `.env.production` - Prod config

## Dockerfile best practices

### Multi-stage builds
```dockerfile
# Stage 1: Build
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
RUN npm run build

# Stage 2: Production
FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY package*.json ./
EXPOSE 3000
USER node
CMD ["node", "dist/main.js"]
```

### Layer caching optimization
```dockerfile
# Copy dependency files first (changes rarely)
COPY package*.json ./
RUN npm ci

# Copy source code last (changes frequently)
COPY . .
RUN npm run build
```

### Security hardening
```dockerfile
# Use specific versions, not 'latest'
FROM node:18.17.1-alpine

# Run as non-root user
USER node

# Use read-only root filesystem
VOLUME /tmp
```

## Docker Compose patterns

### Full-stack application
```yaml
version: '3.8'

services:
  # Frontend
  web:
    build:
      context: ./frontend
      dockerfile: Dockerfile
    ports:
      - "3000:3000"
    environment:
      - API_URL=http://api:8000
    depends_on:
      - api
    volumes:
      - ./frontend:/app
      - /app/node_modules
    networks:
      - app-network

  # Backend API
  api:
    build:
      context: ./backend
      dockerfile: Dockerfile
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=postgresql://user:pass@db:5432/myapp
      - REDIS_URL=redis://cache:6379
    depends_on:
      db:
        condition: service_healthy
      cache:
        condition: service_started
    volumes:
      - ./backend:/app
    networks:
      - app-network

  # PostgreSQL Database
  db:
    image: postgres:15-alpine
    environment:
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=pass
      - POSTGRES_DB=myapp
    ports:
      - "5432:5432"
    volumes:
      - postgres-data:/var/lib/postgresql/data
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U user"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - app-network

  # Redis Cache
  cache:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis-data:/data
    command: redis-server --appendonly yes
    networks:
      - app-network

  # Background Worker
  worker:
    build:
      context: ./backend
      dockerfile: Dockerfile
    command: python worker.py
    environment:
      - DATABASE_URL=postgresql://user:pass@db:5432/myapp
      - REDIS_URL=redis://cache:6379
    depends_on:
      - db
      - cache
    networks:
      - app-network

volumes:
  postgres-data:
  redis-data:

networks:
  app-network:
    driver: bridge
```

### Development overrides
```yaml
# docker-compose.override.yml (auto-loaded in dev)
version: '3.8'

services:
  web:
    volumes:
      - ./frontend:/app
      - /app/node_modules
    environment:
      - NODE_ENV=development

  api:
    volumes:
      - ./backend:/app
    environment:
      - DEBUG=true
    command: python manage.py runserver 0.0.0.0:8000
```

### Production config
```yaml
# docker-compose.prod.yml
version: '3.8'

services:
  web:
    restart: always
    environment:
      - NODE_ENV=production

  api:
    restart: always
    environment:
      - DEBUG=false
    deploy:
      replicas: 3
      resources:
        limits:
          cpus: '0.5'
          memory: 512M
```

## Common service templates

### Node.js app
```yaml
app:
  build: .
  ports:
    - "3000:3000"
  environment:
    - NODE_ENV=development
  volumes:
    - .:/app
    - /app/node_modules
  command: npm run dev
```

### Python/Django app
```yaml
app:
  build: .
  ports:
    - "8000:8000"
  environment:
    - DJANGO_SETTINGS_MODULE=config.settings.development
  volumes:
    - .:/app
  command: python manage.py runserver 0.0.0.0:8000
```

### Go app
```yaml
app:
  build: .
  ports:
    - "8080:8080"
  volumes:
    - .:/app
  command: air  # live reload
```

### Database services
```yaml
# PostgreSQL
postgres:
  image: postgres:15-alpine
  environment:
    POSTGRES_USER: ${DB_USER:-user}
    POSTGRES_PASSWORD: ${DB_PASSWORD:-pass}
    POSTGRES_DB: ${DB_NAME:-myapp}
  volumes:
    - postgres-data:/var/lib/postgresql/data

# MySQL
mysql:
  image: mysql:8
  environment:
    MYSQL_ROOT_PASSWORD: ${DB_ROOT_PASSWORD:-root}
    MYSQL_DATABASE: ${DB_NAME:-myapp}
    MYSQL_USER: ${DB_USER:-user}
    MYSQL_PASSWORD: ${DB_PASSWORD:-pass}
  volumes:
    - mysql-data:/var/lib/mysql

# MongoDB
mongo:
  image: mongo:6
  environment:
    MONGO_INITDB_ROOT_USERNAME: ${DB_USER:-admin}
    MONGO_INITDB_ROOT_PASSWORD: ${DB_PASSWORD:-pass}
  volumes:
    - mongo-data:/data/db
```

### Cache/Queue services
```yaml
# Redis
redis:
  image: redis:7-alpine
  command: redis-server --appendonly yes
  volumes:
    - redis-data:/data

# RabbitMQ
rabbitmq:
  image: rabbitmq:3-management-alpine
  ports:
    - "5672:5672"
    - "15672:15672"  # Management UI
  environment:
    RABBITMQ_DEFAULT_USER: ${RABBIT_USER:-guest}
    RABBITMQ_DEFAULT_PASS: ${RABBIT_PASS:-guest}
```

## .dockerignore template
```
# Version control
.git
.gitignore

# Dependencies
node_modules
vendor
venv
__pycache__

# Environment
.env
.env.local
*.env

# Build artifacts
dist
build
target
*.pyc
*.pyo

# IDE
.vscode
.idea
*.swp

# Testing
coverage
.pytest_cache
.coverage

# OS
.DS_Store
Thumbs.db

# Logs
*.log
logs

# Documentation
README.md
docs
```

## Environment configuration

### .env.example
```bash
# Application
NODE_ENV=development
PORT=3000
API_KEY=your-api-key-here

# Database
DB_HOST=db
DB_PORT=5432
DB_USER=user
DB_PASSWORD=changeme
DB_NAME=myapp

# Redis
REDIS_HOST=cache
REDIS_PORT=6379

# External services
STRIPE_API_KEY=sk_test_xxx
SENDGRID_API_KEY=SG.xxx
```

## Instructions

1. **Analyze project**: Identify language, framework, dependencies
2. **Choose base images**: Official, minimal, specific versions
3. **Create Dockerfile**: Multi-stage, optimized layers
4. **Create docker-compose.yml**: All services with dependencies
5. **Add .dockerignore**: Exclude unnecessary files
6. **Create env files**: Template and development config
7. **Add health checks**: For databases and critical services
8. **Test locally**: `docker-compose up --build`
9. **Document usage**: README with setup instructions

## Useful commands

### Development workflow
```bash
# Start all services
docker-compose up -d

# View logs
docker-compose logs -f [service]

# Rebuild after dependency changes
docker-compose up -d --build

# Run commands in container
docker-compose exec api python manage.py migrate

# Stop all services
docker-compose down

# Stop and remove volumes (fresh start)
docker-compose down -v
```

### Production deployment
```bash
# Build production images
docker-compose -f docker-compose.yml -f docker-compose.prod.yml build

# Deploy
docker-compose -f docker-compose.yml -f docker-compose.prod.yml up -d

# Scale service
docker-compose -f docker-compose.yml -f docker-compose.prod.yml up -d --scale api=3
```

## Output structure
```
project/
  Dockerfile                    # Main app container
  docker-compose.yml            # Development config
  docker-compose.override.yml   # Local overrides (optional)
  docker-compose.prod.yml       # Production config
  .dockerignore                 # Files to exclude
  .env.example                  # Environment template
  scripts/
    entrypoint.sh               # Container startup script
    wait-for-it.sh              # Wait for dependencies
  README.md                     # Setup instructions
```

## README template
````markdown
# Docker Setup

## Prerequisites
- Docker 20.10+
- Docker Compose 2.0+

## Quick Start

1. Copy environment file:
   ```bash
   cp .env.example .env
   ```

2. Start services:
   ```bash
   docker-compose up -d
   ```

3. Run migrations:
   ```bash
   docker-compose exec api python manage.py migrate
   ```

4. Access application:
   - Web: http://localhost:3000
   - API: http://localhost:8000
   - Database: localhost:5432

## Development

### View logs
```bash
docker-compose logs -f
```

### Rebuild after changes
```bash
docker-compose up -d --build
```

### Run tests
```bash
docker-compose exec api pytest
```

## Production

Deploy with:
```bash
docker-compose -f docker-compose.yml -f docker-compose.prod.yml up -d
```
````

## Best practices
- Use specific image versions, not `latest`
- Implement health checks for all services
- Use volumes for persistent data
- Run containers as non-root users
- Keep images small (multi-stage builds, alpine bases)
- Use networks to isolate services
- Environment-specific configs via compose files
- Document service dependencies clearly

## Constraints
- Do NOT commit .env files with secrets
- Do NOT use `latest` tag in production
- Do NOT run containers as root unless necessary
- Always include health checks for databases
- Always use volumes for persistent data
- Test the full stack before considering it complete
