# External Service Integration Tests

## Stripe Payment Integration

### Python + pytest

```python
# tests/integration/test_stripe_payment.py
import pytest
import stripe
from src.services import PaymentService

stripe.api_key = "sk_test_..."  # Test API key


class TestStripePayment:
    def test_create_payment_intent(self):
        """Test creating a payment intent."""
        service = PaymentService()

        payment_intent = service.create_payment(
            amount=1000,  # $10.00
            currency="usd",
            customer_email="test@example.com"
        )

        assert payment_intent.amount == 1000
        assert payment_intent.currency == "usd"
        assert payment_intent.status in ["requires_payment_method", "succeeded"]

    def test_refund_payment(self):
        """Test refunding a payment."""
        service = PaymentService()

        # Create payment
        payment = service.create_payment(amount=2000, currency="usd")

        # Create charge (simulate payment)
        charge = stripe.Charge.create(
            amount=2000,
            currency="usd",
            source="tok_visa",  # Test token
            description="Test charge"
        )

        # Refund
        refund = service.refund_payment(charge.id, amount=1000)

        assert refund.amount == 1000
        assert refund.status == "succeeded"

    def test_webhook_handling(self):
        """Test processing Stripe webhook."""
        service = PaymentService()

        # Simulate webhook payload
        event = {
            "type": "payment_intent.succeeded",
            "data": {
                "object": {
                    "id": "pi_test123",
                    "amount": 3000,
                    "status": "succeeded"
                }
            }
        }

        result = service.handle_webhook(event)

        assert result["status"] == "processed"
        assert result["payment_id"] == "pi_test123"
```

## SendGrid Email Integration

### JavaScript + Jest

```javascript
// tests/integration/email.service.test.js
const sgMail = require('@sendgrid/mail');
const { EmailService } = require('../../src/services/email.service');

sgMail.setApiKey(process.env.SENDGRID_TEST_API_KEY);

describe('EmailService', () => {
  let emailService;

  beforeEach(() => {
    emailService = new EmailService();
  });

  it('should send welcome email', async () => {
    const result = await emailService.sendWelcomeEmail({
      to: 'test@example.com',
      name: 'Test User',
    });

    expect(result.statusCode).toBe(202);
  });

  it('should send password reset email', async () => {
    const result = await emailService.sendPasswordResetEmail({
      to: 'test@example.com',
      resetToken: 'abc123',
    });

    expect(result.statusCode).toBe(202);
  });

  it('should handle invalid email addresses', async () => {
    await expect(
      emailService.sendWelcomeEmail({
        to: 'invalid-email',
        name: 'Test',
      })
    ).rejects.toThrow();
  });
});
```

## AWS S3 Integration

### TypeScript + Vitest

```typescript
// tests/integration/s3.service.test.ts
import { describe, it, expect, beforeAll, afterAll } from 'vitest';
import { S3Client } from '@aws-sdk/client-s3';
import { S3Service } from '../../src/services/s3.service';

const s3Client = new S3Client({
  region: 'us-east-1',
  credentials: {
    accessKeyId: process.env.AWS_TEST_ACCESS_KEY_ID!,
    secretAccessKey: process.env.AWS_TEST_SECRET_ACCESS_KEY!,
  },
});

const s3Service = new S3Service(s3Client);
const TEST_BUCKET = 'test-bucket';

describe('S3Service', () => {
  beforeAll(async () => {
    // Create test bucket
    await s3Service.createBucket(TEST_BUCKET);
  });

  afterAll(async () => {
    // Clean up test bucket
    await s3Service.deleteBucket(TEST_BUCKET);
  });

  it('should upload a file', async () => {
    const content = Buffer.from('Hello, World!');
    const key = 'test-file.txt';

    await s3Service.uploadFile(TEST_BUCKET, key, content);

    const exists = await s3Service.fileExists(TEST_BUCKET, key);
    expect(exists).toBe(true);
  });

  it('should download a file', async () => {
    const content = Buffer.from('Download test');
    const key = 'download-test.txt';

    await s3Service.uploadFile(TEST_BUCKET, key, content);
    const downloaded = await s3Service.downloadFile(TEST_BUCKET, key);

    expect(downloaded.toString()).toBe('Download test');
  });

  it('should delete a file', async () => {
    const key = 'delete-test.txt';
    await s3Service.uploadFile(TEST_BUCKET, key, Buffer.from('Delete me'));

    await s3Service.deleteFile(TEST_BUCKET, key);

    const exists = await s3Service.fileExists(TEST_BUCKET, key);
    expect(exists).toBe(false);
  });

  it('should list files in bucket', async () => {
    await s3Service.uploadFile(TEST_BUCKET, 'file1.txt', Buffer.from('1'));
    await s3Service.uploadFile(TEST_BUCKET, 'file2.txt', Buffer.from('2'));

    const files = await s3Service.listFiles(TEST_BUCKET);

    expect(files.length).toBeGreaterThanOrEqual(2);
  });
});
```

## Auth0 Integration

### Python + pytest

```python
# tests/integration/test_auth0.py
import pytest
import requests
from src.services import Auth0Service

@pytest.fixture
def auth0_service():
    return Auth0Service(
        domain="test-tenant.auth0.com",
        client_id="test_client_id",
        client_secret="test_client_secret"
    )


class TestAuth0Integration:
    def test_create_user(self, auth0_service):
        """Test creating user in Auth0."""
        user = auth0_service.create_user(
            email="newuser@example.com",
            password="SecurePass123!",
            name="New User"
        )

        assert user["email"] == "newuser@example.com"
        assert "user_id" in user

    def test_get_user(self, auth0_service):
        """Test retrieving user from Auth0."""
        created = auth0_service.create_user(
            email="getuser@example.com",
            password="SecurePass123!",
            name="Get User"
        )

        user = auth0_service.get_user(created["user_id"])

        assert user["email"] == "getuser@example.com"

    def test_update_user(self, auth0_service):
        """Test updating user in Auth0."""
        created = auth0_service.create_user(
            email="updateuser@example.com",
            password="SecurePass123!",
            name="Old Name"
        )

        updated = auth0_service.update_user(
            created["user_id"],
            name="New Name"
        )

        assert updated["name"] == "New Name"

    def test_delete_user(self, auth0_service):
        """Test deleting user from Auth0."""
        created = auth0_service.create_user(
            email="deleteuser@example.com",
            password="SecurePass123!",
            name="Delete Me"
        )

        auth0_service.delete_user(created["user_id"])

        # Verify deleted
        with pytest.raises(Exception):
            auth0_service.get_user(created["user_id"])
```

## Testing External APIs with VCR

### Python + vcrpy

```python
# tests/integration/test_github_api.py
import pytest
import vcr
from src.services import GitHubService

@pytest.fixture
def github_service():
    return GitHubService(token="test_token")


@vcr.use_cassette('fixtures/vcr_cassettes/github_user.yaml')
def test_get_user(github_service):
    """Test fetching GitHub user (recorded interaction)."""
    user = github_service.get_user("octocat")

    assert user["login"] == "octocat"
    assert "id" in user


@vcr.use_cassette('fixtures/vcr_cassettes/github_repos.yaml')
def test_list_repos(github_service):
    """Test listing user repositories."""
    repos = github_service.list_repos("octocat")

    assert isinstance(repos, list)
    assert len(repos) > 0
```

VCR records actual HTTP interactions and replays them in future test runs, making tests:
- Faster (no real API calls)
- Deterministic (same responses every time)
- Offline-capable (no network required)
