---
name: integration-test-builder
description: Generates integration tests for REST/GraphQL APIs, database operations, and external service integrations (Stripe, SendGrid, AWS) using pytest (Python), Jest+Supertest (Node.js), or Go testing. Creates tests for CRUD operations, authentication flows, transaction rollbacks, and API contracts. Sets up test databases, fixtures, and Docker containers for isolated testing. Use when testing API endpoints with real databases, verifying external service integrations, testing multi-step workflows, validating database transactions, or bridging unit and E2E test gaps.
allowed-tools: Read, Write, Edit, Bash
---

# Integration Test Builder

You generate integration tests that verify components work together correctly, testing real databases, APIs, and service interactions.

## When to use
- After building API endpoints (test request/response)
- When integrating external services (test real API calls)
- Testing database operations (test queries, transactions)
- Verifying multi-step workflows
- Complementing unit tests with realistic scenarios
- Before deployment to catch integration issues

## What are Integration Tests?

**Unit Tests**: Test individual functions in isolation (mocked dependencies)
**Integration Tests**: Test components working together (real dependencies)
**E2E Tests**: Test entire user flows through UI (slowest, most brittle)

Integration tests sit in the middle:
- Faster than E2E tests
- More realistic than unit tests
- Test actual database/API interactions
- Verify component contracts

## Test Types

### 1. API Integration Tests
Test HTTP endpoints with real requests/responses.

### 2. Database Integration Tests
Test database queries with real database.

### 3. External Service Integration Tests
Test third-party API interactions (Stripe, SendGrid, AWS, etc.)

### 4. Message Queue Integration Tests
Test pub/sub, event-driven systems.

### 5. Workflow Integration Tests
Test multi-step business processes.

## Framework Examples

### REST API Tests - Python (pytest + requests)

**Test file:**
```python
# tests/integration/test_users_api.py
import pytest
import requests

BASE_URL = "http://localhost:8000"

@pytest.fixture
def api_client():
    """Provides authenticated API client."""
    response = requests.post(f"{BASE_URL}/auth/login", json={
        "username": "testuser",
        "password": "testpass123"
    })
    token = response.json()["access_token"]
    return {"Authorization": f"Bearer {token}"}


class TestUsersAPI:
    def test_create_user(self, api_client):
        """Test creating a new user via API."""
        response = requests.post(
            f"{BASE_URL}/users",
            json={"email": "newuser@example.com", "name": "New User"},
            headers=api_client
        )

        assert response.status_code == 201
        data = response.json()
        assert data["email"] == "newuser@example.com"
        assert "id" in data

    def test_get_user(self, api_client):
        """Test retrieving user by ID."""
        # Create user first
        create_response = requests.post(
            f"{BASE_URL}/users",
            json={"email": "getuser@example.com", "name": "Get User"},
            headers=api_client
        )
        user_id = create_response.json()["id"]

        # Retrieve user
        response = requests.get(f"{BASE_URL}/users/{user_id}", headers=api_client)

        assert response.status_code == 200
        data = response.json()
        assert data["id"] == user_id
        assert data["email"] == "getuser@example.com"

    def test_update_user(self, api_client):
        """Test updating user details."""
        # Create user
        create_response = requests.post(
            f"{BASE_URL}/users",
            json={"email": "update@example.com", "name": "Old Name"},
            headers=api_client
        )
        user_id = create_response.json()["id"]

        # Update user
        response = requests.put(
            f"{BASE_URL}/users/{user_id}",
            json={"name": "New Name"},
            headers=api_client
        )

        assert response.status_code == 200
        assert response.json()["name"] == "New Name"

    def test_delete_user(self, api_client):
        """Test deleting a user."""
        # Create user
        create_response = requests.post(
            f"{BASE_URL}/users",
            json={"email": "delete@example.com", "name": "Delete Me"},
            headers=api_client
        )
        user_id = create_response.json()["id"]

        # Delete user
        response = requests.delete(f"{BASE_URL}/users/{user_id}", headers=api_client)
        assert response.status_code == 204

        # Verify deleted
        get_response = requests.get(f"{BASE_URL}/users/{user_id}", headers=api_client)
        assert get_response.status_code == 404

    def test_list_users_pagination(self, api_client):
        """Test listing users with pagination."""
        # Create multiple users
        for i in range(15):
            requests.post(
                f"{BASE_URL}/users",
                json={"email": f"user{i}@example.com", "name": f"User {i}"},
                headers=api_client
            )

        # Test pagination
        response = requests.get(f"{BASE_URL}/users?page=1&limit=10", headers=api_client)

        assert response.status_code == 200
        data = response.json()
        assert len(data["items"]) == 10
        assert data["total"] >= 15
        assert data["page"] == 1
```

### REST API Tests - JavaScript

For JavaScript/TypeScript API tests using Jest + Supertest, including file uploads, WebSocket testing, and GraphQL APIs, see:
- **`examples/javascript-api-tests.md`** - Complete Jest + Supertest examples with advanced patterns

### Database Integration Tests

For database integration tests across multiple languages and ORMs (Python/SQLAlchemy, TypeScript/Prisma, Go/GORM), see:
- **`examples/database-tests.md`** - Complete database test examples with transactions and rollbacks

### External Service Integration Tests

For testing external service integrations (Stripe, SendGrid, AWS S3, Auth0), including VCR for recording API interactions, see:
- **`examples/external-service-tests.md`** - Complete examples for popular third-party services

### Workflow Integration Tests

**Test file:**
```python
# tests/integration/test_order_workflow.py
import pytest
from src.services import OrderService, PaymentService, InventoryService

@pytest.fixture
def services(db_session):
    """Provide all required services."""
    return {
        "order": OrderService(db_session),
        "payment": PaymentService(db_session),
        "inventory": InventoryService(db_session)
    }


class TestOrderWorkflow:
    def test_complete_order_flow(self, services):
        """Test complete order workflow: create → pay → fulfill."""
        order_service = services["order"]
        payment_service = services["payment"]
        inventory_service = services["inventory"]

        # Step 1: Create order
        order = order_service.create_order(
            user_id=1,
            items=[{"product_id": 101, "quantity": 2}]
        )
        assert order.status == "pending"
        assert order.total == 2000  # $20.00

        # Step 2: Process payment
        payment = payment_service.charge(
            order_id=order.id,
            amount=order.total,
            token="tok_visa"
        )
        assert payment.status == "succeeded"

        # Step 3: Update order status
        order = order_service.mark_paid(order.id, payment.id)
        assert order.status == "paid"

        # Step 4: Reserve inventory
        reservation = inventory_service.reserve(order.id, order.items)
        assert reservation.status == "reserved"

        # Step 5: Fulfill order
        order = order_service.fulfill(order.id)
        assert order.status == "fulfilled"

        # Step 6: Verify inventory decreased
        inventory = inventory_service.get_stock(101)
        assert inventory.quantity == (inventory.original_quantity - 2)
```

## Test Setup Patterns

### Using Docker for Test Dependencies

**docker-compose.test.yml:**
```yaml
version: '3.8'
services:
  test-db:
    image: postgres:15-alpine
    environment:
      POSTGRES_USER: test
      POSTGRES_PASSWORD: test
      POSTGRES_DB: test_db
    ports:
      - "5433:5432"

  test-redis:
    image: redis:7-alpine
    ports:
      - "6380:6379"
```

**Run tests:**
```bash
# Start test dependencies
docker-compose -f docker-compose.test.yml up -d

# Run tests
pytest tests/integration

# Cleanup
docker-compose -f docker-compose.test.yml down -v
```

### Using Test Fixtures for Setup/Teardown

**conftest.py:**
```python
import pytest
from src.app import create_app
from src.database import init_db, drop_db

@pytest.fixture(scope="session")
def app():
    """Create application for testing."""
    app = create_app(config="testing")
    return app

@pytest.fixture(scope="function", autouse=True)
def setup_database(app):
    """Setup and teardown database for each test."""
    with app.app_context():
        init_db()
        yield
        drop_db()
```

## Instructions

1. **Identify integration points:**
   - API endpoints to test
   - Database operations
   - External services used
   - Multi-component workflows

2. **Set up test environment:**
   - Test database (separate from dev/prod)
   - Test accounts for external services
   - Docker containers for dependencies

3. **Write integration tests:**
   - Test happy paths first
   - Add error scenarios
   - Test edge cases
   - Cover workflows

4. **Run and verify:**
   - Execute tests locally
   - Check coverage
   - Fix failures
   - Add to CI/CD

## Best Practices

✅ **DO:**
- Use separate test database
- Clean up after each test
- Test both success and failure paths
- Mock external services if flaky
- Run in CI/CD pipeline

❌ **DON'T:**
- Test against production
- Leave test data behind
- Make tests dependent on each other
- Skip error scenarios
- Ignore slow tests (optimize them)

## Constraints

- Must use isolated test environment
- Must clean up test data
- Must be idempotent (can run multiple times)
- Should run in < 5 minutes for full suite
- Must not affect production data
