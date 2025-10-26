# Database Integration Tests

## Python + SQLAlchemy + pytest

### Test File

```python
# tests/integration/test_user_repository.py
import pytest
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker
from src.models import Base, User
from src.repositories import UserRepository

@pytest.fixture(scope="function")
def db_session():
    """Create a test database session."""
    engine = create_engine("postgresql://test:test@localhost/test_db")
    Base.metadata.create_all(engine)

    Session = sessionmaker(bind=engine)
    session = Session()

    yield session

    session.close()
    Base.metadata.drop_all(engine)


class TestUserRepository:
    def test_create_user(self, db_session):
        """Test creating a user in database."""
        repo = UserRepository(db_session)

        user = repo.create(email="test@example.com", name="Test User")

        assert user.id is not None
        assert user.email == "test@example.com"
        assert user.name == "Test User"

    def test_find_by_email(self, db_session):
        """Test finding user by email."""
        repo = UserRepository(db_session)

        # Create user
        created = repo.create(email="find@example.com", name="Find Me")

        # Find user
        found = repo.find_by_email("find@example.com")

        assert found is not None
        assert found.id == created.id

    def test_update_user(self, db_session):
        """Test updating user."""
        repo = UserRepository(db_session)

        user = repo.create(email="update@example.com", name="Old Name")
        updated = repo.update(user.id, name="New Name")

        assert updated.name == "New Name"
        assert updated.email == "update@example.com"

    def test_delete_user(self, db_session):
        """Test deleting user."""
        repo = UserRepository(db_session)

        user = repo.create(email="delete@example.com", name="Delete Me")
        repo.delete(user.id)

        # Verify deleted
        found = repo.find_by_id(user.id)
        assert found is None

    def test_transaction_rollback(self, db_session):
        """Test transaction rollback on error."""
        repo = UserRepository(db_session)

        try:
            db_session.begin_nested()
            repo.create(email="test@example.com", name="User 1")
            repo.create(email="test@example.com", name="User 2")  # Duplicate
            db_session.commit()
        except Exception:
            db_session.rollback()

        # Verify no users created
        users = repo.list_all()
        assert len(users) == 0
```

## TypeScript + Prisma + Vitest

### Setup

```bash
npm install --save-dev vitest @vitest/ui prisma
```

### Test File

```typescript
// tests/integration/user.repository.test.ts
import { describe, it, expect, beforeEach, afterEach } from 'vitest';
import { PrismaClient } from '@prisma/client';
import { UserRepository } from '../../src/repositories/user.repository';

const prisma = new PrismaClient();
const userRepo = new UserRepository(prisma);

describe('UserRepository', () => {
  beforeEach(async () => {
    // Clean database before each test
    await prisma.user.deleteMany();
  });

  afterEach(async () => {
    // Clean up after each test
    await prisma.user.deleteMany();
  });

  it('should create a user', async () => {
    const user = await userRepo.create({
      email: 'test@example.com',
      name: 'Test User',
    });

    expect(user.id).toBeDefined();
    expect(user.email).toBe('test@example.com');
    expect(user.name).toBe('Test User');
  });

  it('should find user by email', async () => {
    await userRepo.create({
      email: 'find@example.com',
      name: 'Find Me',
    });

    const user = await userRepo.findByEmail('find@example.com');

    expect(user).toBeDefined();
    expect(user?.email).toBe('find@example.com');
  });

  it('should update user', async () => {
    const created = await userRepo.create({
      email: 'update@example.com',
      name: 'Old Name',
    });

    const updated = await userRepo.update(created.id, {
      name: 'New Name',
    });

    expect(updated.name).toBe('New Name');
    expect(updated.email).toBe('update@example.com');
  });

  it('should delete user', async () => {
    const created = await userRepo.create({
      email: 'delete@example.com',
      name: 'Delete Me',
    });

    await userRepo.delete(created.id);

    const found = await userRepo.findById(created.id);
    expect(found).toBeNull();
  });

  it('should handle transaction rollback', async () => {
    await expect(async () => {
      await prisma.$transaction(async (tx) => {
        await tx.user.create({
          data: { email: 'tx1@example.com', name: 'User 1' },
        });
        await tx.user.create({
          data: { email: 'tx1@example.com', name: 'User 2' }, // Duplicate
        });
      });
    }).rejects.toThrow();

    const users = await prisma.user.findMany();
    expect(users).toHaveLength(0);
  });
});
```

## Go + GORM + Testing

### Test File

```go
// repository_test.go
package repository_test

import (
	"testing"
	"gorm.io/driver/sqlite"
	"gorm.io/gorm"
	"github.com/stretchr/testify/assert"
)

func setupTestDB(t *testing.T) *gorm.DB {
	db, err := gorm.Open(sqlite.Open(":memory:"), &gorm.Config{})
	assert.NoError(t, err)

	err = db.AutoMigrate(&User{})
	assert.NoError(t, err)

	return db
}

func TestUserRepository_Create(t *testing.T) {
	db := setupTestDB(t)
	repo := NewUserRepository(db)

	user := &User{
		Email: "test@example.com",
		Name:  "Test User",
	}

	err := repo.Create(user)
	assert.NoError(t, err)
	assert.NotZero(t, user.ID)
}

func TestUserRepository_FindByEmail(t *testing.T) {
	db := setupTestDB(t)
	repo := NewUserRepository(db)

	// Create user
	user := &User{Email: "find@example.com", Name: "Find Me"}
	repo.Create(user)

	// Find user
	found, err := repo.FindByEmail("find@example.com")
	assert.NoError(t, err)
	assert.NotNil(t, found)
	assert.Equal(t, user.ID, found.ID)
}

func TestUserRepository_Update(t *testing.T) {
	db := setupTestDB(t)
	repo := NewUserRepository(db)

	user := &User{Email: "update@example.com", Name: "Old Name"}
	repo.Create(user)

	user.Name = "New Name"
	err := repo.Update(user)
	assert.NoError(t, err)

	found, _ := repo.FindByID(user.ID)
	assert.Equal(t, "New Name", found.Name)
}

func TestUserRepository_Delete(t *testing.T) {
	db := setupTestDB(t)
	repo := NewUserRepository(db)

	user := &User{Email: "delete@example.com", Name: "Delete Me"}
	repo.Create(user)

	err := repo.Delete(user.ID)
	assert.NoError(t, err)

	found, _ := repo.FindByID(user.ID)
	assert.Nil(t, found)
}
```
