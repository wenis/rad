# JavaScript API Integration Tests

## Using Jest + Supertest

### Setup

```bash
npm install --save-dev jest supertest
```

### Complete Test Suite

```javascript
// tests/integration/users.test.js
const request = require('supertest');
const app = require('../../src/app');

describe('Users API', () => {
  let authToken;
  let userId;

  beforeAll(async () => {
    // Get auth token
    const response = await request(app)
      .post('/auth/login')
      .send({ username: 'testuser', password: 'testpass123' });
    authToken = response.body.accessToken;
  });

  describe('POST /users', () => {
    it('should create a new user', async () => {
      const response = await request(app)
        .post('/users')
        .set('Authorization', `Bearer ${authToken}`)
        .send({ email: 'newuser@example.com', name: 'New User' })
        .expect(201);

      expect(response.body).toHaveProperty('id');
      expect(response.body.email).toBe('newuser@example.com');
      userId = response.body.id;
    });

    it('should reject duplicate email', async () => {
      await request(app)
        .post('/users')
        .set('Authorization', `Bearer ${authToken}`)
        .send({ email: 'newuser@example.com', name: 'Duplicate' })
        .expect(409);
    });

    it('should reject invalid email', async () => {
      const response = await request(app)
        .post('/users')
        .set('Authorization', `Bearer ${authToken}`)
        .send({ email: 'invalid-email', name: 'Invalid' })
        .expect(400);

      expect(response.body).toHaveProperty('errors');
      expect(response.body.errors).toContain('email');
    });
  });

  describe('GET /users/:id', () => {
    it('should retrieve user by ID', async () => {
      const response = await request(app)
        .get(`/users/${userId}`)
        .set('Authorization', `Bearer ${authToken}`)
        .expect(200);

      expect(response.body.id).toBe(userId);
      expect(response.body.email).toBe('newuser@example.com');
    });

    it('should return 404 for non-existent user', async () => {
      await request(app)
        .get('/users/99999')
        .set('Authorization', `Bearer ${authToken}`)
        .expect(404);
    });
  });

  describe('PUT /users/:id', () => {
    it('should update user', async () => {
      const response = await request(app)
        .put(`/users/${userId}`)
        .set('Authorization', `Bearer ${authToken}`)
        .send({ name: 'Updated Name' })
        .expect(200);

      expect(response.body.name).toBe('Updated Name');
    });
  });

  describe('DELETE /users/:id', () => {
    it('should delete user', async () => {
      await request(app)
        .delete(`/users/${userId}`)
        .set('Authorization', `Bearer ${authToken}`)
        .expect(204);

      // Verify deleted
      await request(app)
        .get(`/users/${userId}`)
        .set('Authorization', `Bearer ${authToken}`)
        .expect(404);
    });
  });
});
```

## Advanced Patterns

### Testing File Uploads

```javascript
const fs = require('fs');
const path = require('path');

describe('File Upload', () => {
  it('should upload a profile picture', async () => {
    const filePath = path.join(__dirname, 'fixtures', 'avatar.jpg');

    const response = await request(app)
      .post('/users/1/avatar')
      .set('Authorization', `Bearer ${authToken}`)
      .attach('file', filePath)
      .expect(200);

    expect(response.body).toHaveProperty('avatarUrl');
  });
});
```

### Testing WebSocket Connections

```javascript
const io = require('socket.io-client');

describe('WebSocket API', () => {
  let socket;

  beforeEach((done) => {
    socket = io('http://localhost:3000', {
      auth: { token: authToken }
    });
    socket.on('connect', done);
  });

  afterEach(() => {
    socket.disconnect();
  });

  it('should receive real-time updates', (done) => {
    socket.on('user:created', (data) => {
      expect(data).toHaveProperty('id');
      expect(data.email).toBe('realtime@example.com');
      done();
    });

    // Trigger event
    request(app)
      .post('/users')
      .set('Authorization', `Bearer ${authToken}`)
      .send({ email: 'realtime@example.com', name: 'Realtime User' });
  });
});
```

### Testing GraphQL APIs

```javascript
describe('GraphQL API', () => {
  it('should query users', async () => {
    const query = `
      query {
        users {
          id
          email
          name
        }
      }
    `;

    const response = await request(app)
      .post('/graphql')
      .set('Authorization', `Bearer ${authToken}`)
      .send({ query })
      .expect(200);

    expect(response.body.data.users).toBeInstanceOf(Array);
  });

  it('should create user via mutation', async () => {
    const mutation = `
      mutation CreateUser($input: CreateUserInput!) {
        createUser(input: $input) {
          id
          email
          name
        }
      }
    `;

    const variables = {
      input: {
        email: 'graphql@example.com',
        name: 'GraphQL User'
      }
    };

    const response = await request(app)
      .post('/graphql')
      .set('Authorization', `Bearer ${authToken}`)
      .send({ query: mutation, variables })
      .expect(200);

    expect(response.body.data.createUser).toHaveProperty('id');
  });
});
```
