---
name: database-migration-manager
description: Creates, manages, and validates database migrations for schema changes. Handles migrations for PostgreSQL, MySQL, MongoDB. Use when adding/modifying database tables OR creating migrations OR planning schema changes OR investigating migration failures.
allowed-tools: Read, Write, Edit, Bash
---

# Database Migration Manager

You create and manage database migrations safely across development, staging, and production environments.

## When to use
- Adding new database tables or columns
- Modifying existing schema (add indexes, change types)
- Creating data migrations (backfill, transform)
- Rolling back failed migrations
- Planning database changes for a feature
- Investigating migration conflicts

## Supported Frameworks

### Python
- **Django** - Built-in migrations
- **Alembic** (SQLAlchemy) - Popular for Flask, FastAPI
- **Peewee** - Lightweight ORM

### JavaScript/TypeScript
- **Prisma** - Modern ORM with migrations
- **TypeORM** - Enterprise TypeScript ORM
- **Knex.js** - SQL query builder with migrations
- **Sequelize** - Traditional Node.js ORM

### Go
- **golang-migrate** - Database-agnostic migrations
- **Goose** - Go migration tool

### Ruby
- **ActiveRecord** (Rails) - Ruby on Rails migrations

## Migration Principles

### 1. Backward Compatibility
Always write migrations that work with old code:
- Add columns as nullable first, then backfill, then make required
- Don't remove columns until old code is fully deployed
- Use multi-step migrations for breaking changes

### 2. Reversibility
Every migration must be reversible:
- Provide `down()` or `reverse()` method
- Test rollback before deploying
- Some migrations can't be reversed (data deletion) - document this

### 3. Idempotency
Migrations should be safe to run multiple times:
- Check if change already exists before applying
- Use `IF NOT EXISTS` or equivalent
- Handle partial completion gracefully

### 4. Data Safety
Never lose data:
- Backup before destructive operations
- Use transactions where possible
- Test on staging first
- Have rollback plan ready

## Migration Patterns

### Adding a Column (Safe Pattern)

**Bad (breaks old code):**
```sql
ALTER TABLE users ADD COLUMN email VARCHAR(255) NOT NULL;
```

**Good (backward compatible):**
```python
# Migration 1: Add nullable column
def up():
    op.add_column('users', sa.Column('email', sa.String(255), nullable=True))

def down():
    op.drop_column('users', 'email')
```

```python
# Migration 2 (after old code deployed): Backfill data
def up():
    op.execute("UPDATE users SET email = username || '@example.com' WHERE email IS NULL")

def down():
    op.execute("UPDATE users SET email = NULL")
```

```python
# Migration 3 (after backfill): Make NOT NULL
def up():
    op.alter_column('users', 'email', nullable=False)

def down():
    op.alter_column('users', 'email', nullable=True)
```

### Renaming a Column (Multi-Step)

**Step 1: Add new column**
```python
def up():
    op.add_column('users', sa.Column('full_name', sa.String(255), nullable=True))

def down():
    op.drop_column('users', 'full_name')
```

**Step 2: Dual write (code changes)**
```python
# Application code writes to both old and new columns
user.name = "John Doe"
user.full_name = "John Doe"
```

**Step 3: Backfill**
```python
def up():
    op.execute("UPDATE users SET full_name = name WHERE full_name IS NULL")

def down():
    op.execute("UPDATE users SET full_name = NULL")
```

**Step 4: Switch reads to new column** (code change)

**Step 5: Drop old column**
```python
def up():
    op.drop_column('users', 'name')

def down():
    op.add_column('users', sa.Column('name', sa.String(255), nullable=True))
```

### Adding an Index (No Downtime)

**PostgreSQL:**
```python
def up():
    op.create_index('idx_users_email', 'users', ['email'],
                   postgresql_concurrently=True)

def down():
    op.drop_index('idx_users_email', 'users')
```

**MySQL:**
```python
def up():
    op.execute("ALTER TABLE users ADD INDEX idx_users_email (email) ALGORITHM=INPLACE, LOCK=NONE")

def down():
    op.drop_index('idx_users_email', 'users')
```

## Framework-Specific Examples

### Alembic (Python/SQLAlchemy)

**Generate migration:**
```bash
alembic revision -m "add users email column"
```

**Migration file:**
```python
"""add users email column

Revision ID: abc123
Revises: xyz789
Create Date: 2025-01-15
"""
from alembic import op
import sqlalchemy as sa

revision = 'abc123'
down_revision = 'xyz789'

def upgrade():
    op.add_column('users',
        sa.Column('email', sa.String(255), nullable=True)
    )
    op.create_index('idx_users_email', 'users', ['email'])

def downgrade():
    op.drop_index('idx_users_email', 'users')
    op.drop_column('users', 'email')
```

**Run migration:**
```bash
alembic upgrade head
```

**Rollback:**
```bash
alembic downgrade -1
```

### Prisma (TypeScript)

**Schema change:**
```prisma
model User {
  id        Int      @id @default(autoincrement())
  email     String   @unique  // Added this
  name      String
  createdAt DateTime @default(now())
}
```

**Generate migration:**
```bash
npx prisma migrate dev --name add_user_email
```

**Generated migration (SQL):**
```sql
-- CreateIndex
CREATE UNIQUE INDEX "User_email_key" ON "User"("email");

-- AlterTable
ALTER TABLE "User" ADD COLUMN "email" TEXT NOT NULL;
```

**Deploy to production:**
```bash
npx prisma migrate deploy
```

### TypeORM (TypeScript)

**Generate migration:**
```bash
npm run typeorm migration:generate -- -n AddUserEmail
```

**Migration file:**
```typescript
import { MigrationInterface, QueryRunner, TableColumn } from "typeorm";

export class AddUserEmail1234567890 implements MigrationInterface {
    public async up(queryRunner: QueryRunner): Promise<void> {
        await queryRunner.addColumn("users", new TableColumn({
            name: "email",
            type: "varchar",
            length: "255",
            isNullable: true
        }));

        await queryRunner.createIndex("users", {
            name: "IDX_USER_EMAIL",
            columnNames: ["email"]
        });
    }

    public async down(queryRunner: QueryRunner): Promise<void> {
        await queryRunner.dropIndex("users", "IDX_USER_EMAIL");
        await queryRunner.dropColumn("users", "email");
    }
}
```

**Run migration:**
```bash
npm run typeorm migration:run
```

### Django (Python)

**Generate migration:**
```bash
python manage.py makemigrations
```

**Migration file:**
```python
from django.db import migrations, models

class Migration(migrations.Migration):
    dependencies = [
        ('users', '0001_initial'),
    ]

    operations = [
        migrations.AddField(
            model_name='user',
            name='email',
            field=models.EmailField(max_length=255, null=True),
        ),
        migrations.AddIndex(
            model_name='user',
            index=models.Index(fields=['email'], name='users_email_idx'),
        ),
    ]
```

**Run migration:**
```bash
python manage.py migrate
```

## Migration Workflow

### 1. Plan the Migration

**Analyze impact:**
- Will this break existing code?
- Is data transformation needed?
- Can we do this online (no downtime)?
- How much data is affected?

**Create plan:**
- Single migration or multi-step?
- Need to backfill data?
- What's the rollback strategy?
- Test on staging first?

### 2. Write the Migration

**Include:**
- Clear description in migration name
- Both up and down methods
- Indexes for new columns (if queried)
- Data validations
- Comments for complex logic

**Avoid:**
- Accessing application models (they change)
- Long-running operations without batching
- Mixing DDL and DML in same transaction (MySQL)

### 3. Test the Migration

**Local testing:**
```bash
# Run migration
npm run migrate:up

# Verify schema
psql -d myapp_dev -c "\d users"

# Test rollback
npm run migrate:down

# Verify rollback worked
psql -d myapp_dev -c "\d users"

# Re-run migration
npm run migrate:up
```

**Staging testing:**
```bash
# Deploy to staging
git push staging main

# Run migration on staging
ssh staging "cd /app && npm run migrate"

# Verify application works
curl https://staging.example.com/health

# Check for errors
tail -f /var/log/app.log
```

### 4. Deploy to Production

**Pre-deployment:**
- Backup database
- Run in maintenance window if needed
- Have rollback plan ready
- Monitor tools prepared

**Deployment:**
```bash
# Run migration
npm run migrate:production

# Verify schema
psql -d myapp_prod -c "\d users"

# Monitor application
tail -f /var/log/app.log

# Check error rates
curl https://api.example.com/metrics
```

**Post-deployment:**
- Monitor for 30 minutes
- Check error rates and performance
- Verify new feature works
- Keep rollback ready

## Common Issues and Solutions

### Issue: Migration hanging

**Cause:** Waiting for lock on table

**Solution:**
```sql
-- Check for locks
SELECT * FROM pg_locks WHERE NOT granted;

-- Find blocking queries
SELECT pid, query FROM pg_stat_activity WHERE state = 'active';

-- Kill blocking query (if safe)
SELECT pg_terminate_backend(12345);
```

### Issue: Out of order migrations

**Cause:** Multiple branches creating migrations

**Solution:**
```bash
# Regenerate migration with correct parent
alembic revision --head xyz789 -m "add email column"

# Or merge branches
alembic merge head1 head2 -m "merge migrations"
```

### Issue: Data too large to migrate at once

**Solution:** Batch processing
```python
def upgrade():
    # Batch update in chunks
    batch_size = 1000
    offset = 0

    while True:
        result = op.execute(f"""
            UPDATE users
            SET email = username || '@example.com'
            WHERE email IS NULL
            LIMIT {batch_size}
            OFFSET {offset}
        """)

        if result.rowcount == 0:
            break

        offset += batch_size
        print(f"Processed {offset} rows")
```

## Instructions

1. **Understand the schema change:**
   - Read spec or feature requirements
   - Identify what tables/columns are affected
   - Determine if data migration needed

2. **Determine migration strategy:**
   - Breaking change? → Multi-step migration
   - Data transformation? → Separate data migration
   - Large dataset? → Batch processing

3. **Write migration(s):**
   - Use appropriate framework command
   - Include up and down methods
   - Add comments for complex logic

4. **Test thoroughly:**
   - Run locally → verify → rollback → re-run
   - Deploy to staging → test application → check logs

5. **Create deployment plan:**
   - Backup strategy
   - Rollback plan
   - Monitoring checklist

6. **Document:**
   - Add comments in migration file
   - Note any manual steps required
   - Document rollback procedure if non-standard

## Output Artifacts

- Migration files (generated)
- Migration plan document (for complex changes)
- Rollback script (if needed)
- Post-migration validation checklist

## Best Practices

✅ **DO:**
- Write reversible migrations
- Test rollback before deploying
- Use transactions when possible
- Add indexes for new query patterns
- Batch large data migrations
- Document breaking changes
- Run on staging first

❌ **DON'T:**
- Mix schema and data changes
- Access application models in migrations
- Run without backup on production
- Assume migrations are fast
- Ignore rollback scenarios
- Skip staging testing

## Constraints

- Never run destructive migrations without backup
- Always test rollback functionality
- Document irreversible migrations clearly
- Use transactions unless database doesn't support them
- Test on realistic data volumes in staging
