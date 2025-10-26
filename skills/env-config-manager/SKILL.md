---
name: env-config-manager
description: Manages environment variables and configuration across dev/staging/prod using .env files, dotenv, docker-compose env vars, or cloud secret managers (AWS Secrets Manager, Azure Key Vault). Creates .env.example templates, validates required vars, implements type-safe config loading, and migrates hardcoded config to environment variables. Handles secrets securely without committing them. Use when setting up deployment configs, managing API keys/secrets, creating environment templates, containerizing apps, or migrating to 12-factor app patterns.
allowed-tools: Read, Write, Edit
---

# Environment Config Manager

You manage environment-based configuration, keeping secrets secure and configs maintainable across environments.

## When to use
- Setting up a new project's configuration
- Migrating hardcoded config to environment variables
- Creating environment templates for team onboarding
- Managing multi-environment deployments (dev/staging/prod)
- Implementing 12-factor app principles
- Auditing for hardcoded secrets

## Configuration principles

### 1. Never commit secrets
- API keys, passwords, tokens → environment variables
- Use `.env.example` as template with dummy values
- Add `.env*` to `.gitignore` (except `.env.example`)

### 2. Environment parity
- Same variable names across all environments
- Only values differ between dev/staging/prod
- Document all required variables

### 3. Sensible defaults
- Provide defaults for non-sensitive config
- Make local development work out-of-box
- Fail fast on missing critical config

### 4. Type safety
- Validate environment variables at startup
- Provide clear error messages for invalid config
- Use schema validation where possible

## File structure

```
project/
  .env.example           # Template (committed)
  .env                   # Local development (not committed)
  .env.development       # Development defaults (optional, committed)
  .env.staging           # Staging values (not committed)
  .env.production        # Production values (not committed)
  .env.test              # Testing config (committed)
  config/
    env.ts               # Environment loading/validation
    secrets.md           # Documentation for secret setup
```

## .env.example template

```bash
# =============================================================================
# APPLICATION CONFIGURATION
# Copy this file to .env and fill in the values
# =============================================================================

# -------------------------------------
# Application
# -------------------------------------
NODE_ENV=development
PORT=3000
APP_NAME=MyApp
LOG_LEVEL=debug

# -------------------------------------
# Database
# -------------------------------------
DATABASE_URL=postgresql://user:password@localhost:5432/myapp_dev
DB_POOL_SIZE=10

# -------------------------------------
# Redis Cache
# -------------------------------------
REDIS_URL=redis://localhost:6379
REDIS_TTL=3600

# -------------------------------------
# Authentication
# -------------------------------------
JWT_SECRET=your-secret-key-change-me
JWT_EXPIRES_IN=24h
SESSION_SECRET=your-session-secret-change-me

# -------------------------------------
# External APIs
# -------------------------------------
# Stripe (get from: https://dashboard.stripe.com/apikeys)
STRIPE_SECRET_KEY=sk_test_xxx
STRIPE_PUBLISHABLE_KEY=pk_test_xxx
STRIPE_WEBHOOK_SECRET=whsec_xxx

# SendGrid (get from: https://app.sendgrid.com/settings/api_keys)
SENDGRID_API_KEY=SG.xxx
FROM_EMAIL=noreply@example.com

# AWS S3
AWS_ACCESS_KEY_ID=AKIA...
AWS_SECRET_ACCESS_KEY=xxx
AWS_REGION=us-east-1
AWS_S3_BUCKET=my-app-uploads

# -------------------------------------
# Feature Flags
# -------------------------------------
ENABLE_ANALYTICS=true
ENABLE_EMAIL_NOTIFICATIONS=true
MAINTENANCE_MODE=false

# -------------------------------------
# URLs
# -------------------------------------
API_URL=http://localhost:8000
FRONTEND_URL=http://localhost:3000
CALLBACK_URL=http://localhost:3000/auth/callback
```

## Environment-specific configs

### Development (.env or .env.development)
```bash
NODE_ENV=development
PORT=3000
DATABASE_URL=postgresql://user:pass@localhost:5432/myapp_dev
REDIS_URL=redis://localhost:6379
LOG_LEVEL=debug
DEBUG=true
```

### Staging (.env.staging - example, not committed)
```bash
NODE_ENV=staging
PORT=8080
DATABASE_URL=postgresql://user:pass@db-staging.example.com:5432/myapp_staging
REDIS_URL=redis://cache-staging.example.com:6379
LOG_LEVEL=info
DEBUG=false
```

### Production (.env.production - example, not committed)
```bash
NODE_ENV=production
PORT=8080
DATABASE_URL=postgresql://user:pass@db-prod.example.com:5432/myapp
REDIS_URL=redis://cache-prod.example.com:6379
LOG_LEVEL=warn
DEBUG=false
ENABLE_ANALYTICS=true
```

### Testing (.env.test)
```bash
NODE_ENV=test
DATABASE_URL=postgresql://user:pass@localhost:5432/myapp_test
REDIS_URL=redis://localhost:6379/1
LOG_LEVEL=error
ENABLE_EMAIL_NOTIFICATIONS=false
```

## Configuration loading

### TypeScript/Node.js
```typescript
// config/env.ts
import dotenv from 'dotenv';
import { z } from 'zod';

// Load .env file
dotenv.config();

// Define schema
const envSchema = z.object({
  NODE_ENV: z.enum(['development', 'staging', 'production', 'test']),
  PORT: z.string().transform(Number),
  DATABASE_URL: z.string().url(),
  REDIS_URL: z.string().url(),
  JWT_SECRET: z.string().min(32),
  STRIPE_SECRET_KEY: z.string().startsWith('sk_'),
  LOG_LEVEL: z.enum(['debug', 'info', 'warn', 'error']).default('info'),
});

// Validate and export
export const env = envSchema.parse(process.env);

// Type-safe access
// env.PORT is number
// env.NODE_ENV is 'development' | 'staging' | 'production' | 'test'
```

Usage:
```typescript
import { env } from './config/env';

console.log(`Server running on port ${env.PORT}`);
// TypeScript knows env.PORT is a number
```

### Python
```python
# config/settings.py
import os
from typing import Literal
from pydantic import BaseSettings, Field, validator


class Settings(BaseSettings):
    # Application
    node_env: Literal['development', 'staging', 'production', 'test'] = 'development'
    port: int = 3000
    log_level: str = 'info'

    # Database
    database_url: str
    db_pool_size: int = 10

    # Redis
    redis_url: str

    # Authentication
    jwt_secret: str = Field(..., min_length=32)
    jwt_expires_in: str = '24h'

    # External APIs
    stripe_secret_key: str
    sendgrid_api_key: str
    from_email: str

    @validator('stripe_secret_key')
    def validate_stripe_key(cls, v):
        if not v.startswith('sk_'):
            raise ValueError('Invalid Stripe secret key format')
        return v

    class Config:
        env_file = '.env'
        case_sensitive = False


settings = Settings()
```

Usage:
```python
from config.settings import settings

print(f"Server running on port {settings.port}")
```

### Go
```go
// config/env.go
package config

import (
    "fmt"
    "os"
    "strconv"

    "github.com/joho/godotenv"
)

type Config struct {
    NodeEnv     string
    Port        int
    DatabaseURL string
    RedisURL    string
    JWTSecret   string
    LogLevel    string
}

func Load() (*Config, error) {
    // Load .env file
    godotenv.Load()

    port, err := strconv.Atoi(getEnv("PORT", "3000"))
    if err != nil {
        return nil, fmt.Errorf("invalid PORT: %w", err)
    }

    config := &Config{
        NodeEnv:     getEnv("NODE_ENV", "development"),
        Port:        port,
        DatabaseURL: mustGetEnv("DATABASE_URL"),
        RedisURL:    mustGetEnv("REDIS_URL"),
        JWTSecret:   mustGetEnv("JWT_SECRET"),
        LogLevel:    getEnv("LOG_LEVEL", "info"),
    }

    return config, nil
}

func getEnv(key, fallback string) string {
    if value, ok := os.LookupEnv(key); ok {
        return value
    }
    return fallback
}

func mustGetEnv(key string) string {
    value, ok := os.LookupEnv(key)
    if !ok {
        panic(fmt.Sprintf("Required environment variable %s not set", key))
    }
    return value
}
```

## Secret management best practices

### Local development
```bash
# Option 1: .env file (simplest)
cp .env.example .env
# Edit .env with local values

# Option 2: direnv (auto-loads on cd)
echo "export DATABASE_URL=..." > .envrc
direnv allow

# Option 3: Shell profile
echo "export DATABASE_URL=..." >> ~/.zshrc
```

### CI/CD
```yaml
# GitHub Actions
env:
  DATABASE_URL: ${{ secrets.DATABASE_URL }}
  STRIPE_SECRET_KEY: ${{ secrets.STRIPE_SECRET_KEY }}

# GitLab CI
variables:
  DATABASE_URL: $DATABASE_URL
  STRIPE_SECRET_KEY: $STRIPE_SECRET_KEY
```

### Production
Use dedicated secret management:
- **AWS**: AWS Secrets Manager, Parameter Store
- **GCP**: Secret Manager
- **Azure**: Key Vault
- **Kubernetes**: Secrets
- **HashiCorp**: Vault
- **Doppler**: Config management platform

## Documentation template

### secrets.md
```markdown
# Environment Variables Setup

## Required Variables

### DATABASE_URL
PostgreSQL connection string.

**Format:** `postgresql://user:password@host:port/database`

**Local development:**
```bash
DATABASE_URL=postgresql://localhost:5432/myapp_dev
```

**Production:** Use managed database URL from cloud provider.

### STRIPE_SECRET_KEY
Stripe API secret key for payment processing.

**How to get:**
1. Log in to [Stripe Dashboard](https://dashboard.stripe.com)
2. Go to Developers → API Keys
3. Copy "Secret key" (starts with `sk_`)

**Environments:**
- Development: Use test key (`sk_test_...`)
- Production: Use live key (`sk_live_...`)

### JWT_SECRET
Secret key for signing JWT tokens.

**How to generate:**
```bash
openssl rand -hex 32
```

**Important:** Use different secrets for each environment!

## Optional Variables

### LOG_LEVEL
Controls logging verbosity.

**Options:** `debug`, `info`, `warn`, `error`

**Default:** `info`

## Setup Instructions

1. Copy template:
   ```bash
   cp .env.example .env
   ```

2. Fill in required values in `.env`

3. Verify configuration:
   ```bash
   npm run check-env
   ```
```

## .gitignore additions
```
# Environment files
.env
.env.local
.env.*.local
.env.staging
.env.production

# Keep templates
!.env.example
!.env.test
```

## Validation script

### package.json
```json
{
  "scripts": {
    "check-env": "node scripts/check-env.js"
  }
}
```

### scripts/check-env.js
```javascript
const required = [
  'DATABASE_URL',
  'REDIS_URL',
  'JWT_SECRET',
  'STRIPE_SECRET_KEY'
];

const missing = required.filter(key => !process.env[key]);

if (missing.length > 0) {
  console.error('❌ Missing required environment variables:');
  missing.forEach(key => console.error(`  - ${key}`));
  process.exit(1);
}

console.log('✅ All required environment variables are set');
```

## Instructions

1. **Audit existing config**: Find all hardcoded values
2. **Create .env.example**: Template with all variables
3. **Implement loading**: Use library (dotenv, pydantic, etc.)
4. **Add validation**: Schema checking at startup
5. **Document secrets**: How to obtain each value
6. **Update .gitignore**: Exclude real .env files
7. **Test environments**: Verify dev/staging/prod configs

## Best practices
- Use descriptive variable names (DATABASE_URL, not DB)
- Group related variables with comments
- Provide working defaults for local development
- Document where to get secret values
- Validate required variables at startup
- Use different secrets for each environment
- Never log sensitive variable values
- Rotate secrets regularly

## Constraints
- Do NOT commit .env files with real secrets
- Do NOT use production secrets in development
- Always validate required variables at startup
- Always document how to obtain secret values
- Keep .env.example up-to-date with code changes
