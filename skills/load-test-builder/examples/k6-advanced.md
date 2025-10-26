# k6 Advanced Examples

## POST with Authentication

```javascript
import http from 'k6/http';
import { check } from 'k6';

export const options = {
  stages: [
    { duration: '30s', target: 20 },
    { duration: '1m', target: 20 },
    { duration: '30s', target: 0 },
  ],
};

// Setup: Run once per VU
export function setup() {
  // Login and get token
  const loginRes = http.post('https://api.example.com/auth/login', {
    email: 'test@example.com',
    password: 'password123',
  });

  return { token: loginRes.json('access_token') };
}

export default function (data) {
  const params = {
    headers: {
      'Content-Type': 'application/json',
      'Authorization': `Bearer ${data.token}`,
    },
  };

  // Create user
  const payload = JSON.stringify({
    email: 'user@example.com',
    name: 'Test User',
  });

  const res = http.post('https://api.example.com/users', payload, params);

  check(res, {
    'status is 201': (r) => r.status === 201,
    'user created': (r) => r.json('id') !== undefined,
  });
}
```

## Multiple Endpoints (Realistic Scenario)

```javascript
import http from 'k6/http';
import { check, group, sleep } from 'k6';
import { randomIntBetween } from 'https://jslib.k6.io/k6-utils/1.2.0/index.js';

export const options = {
  stages: [
    { duration: '5m', target: 100 },
    { duration: '10m', target: 100 },
    { duration: '5m', target: 0 },
  ],
  thresholds: {
    http_req_duration: ['p(95)<500'],
    'http_req_duration{endpoint:list_users}': ['p(95)<200'],
    'http_req_duration{endpoint:get_user}': ['p(95)<100'],
    'http_req_duration{endpoint:create_user}': ['p(95)<300'],
  },
};

const BASE_URL = 'https://api.example.com';

export default function () {
  // 60% read, 40% write (realistic ratio)
  const isRead = Math.random() < 0.6;

  if (isRead) {
    group('Read Operations', function () {
      // List users
      let res = http.get(`${BASE_URL}/users`, {
        tags: { endpoint: 'list_users' },
      });
      check(res, { 'list status 200': (r) => r.status === 200 });

      // Get specific user
      const userId = randomIntBetween(1, 1000);
      res = http.get(`${BASE_URL}/users/${userId}`, {
        tags: { endpoint: 'get_user' },
      });
      check(res, { 'get status 200': (r) => r.status === 200 });
    });
  } else {
    group('Write Operations', function () {
      // Create user
      const payload = JSON.stringify({
        email: `user${Date.now()}@example.com`,
        name: `User ${Date.now()}`,
      });

      const res = http.post(`${BASE_URL}/users`, payload, {
        headers: { 'Content-Type': 'application/json' },
        tags: { endpoint: 'create_user' },
      });

      check(res, { 'create status 201': (r) => r.status === 201 });
    });
  }

  sleep(randomIntBetween(1, 3));
}
```

## Stress Test (Find Breaking Point)

```javascript
export const options = {
  stages: [
    { duration: '2m', target: 100 },   // Warm up
    { duration: '5m', target: 100 },   // Stay at 100
    { duration: '2m', target: 200 },   // Bump to 200
    { duration: '5m', target: 200 },   // Stay at 200
    { duration: '2m', target: 300 },   // Bump to 300
    { duration: '5m', target: 300 },   // Stay at 300
    { duration: '2m', target: 400 },   // Push to 400
    { duration: '5m', target: 400 },   // Stay at 400
    { duration: '10m', target: 0 },    // Ramp down
  ],
  thresholds: {
    http_req_duration: ['p(99)<1000'],  // More lenient for stress test
    http_req_failed: ['rate<0.05'],     // Allow 5% error rate
  },
};
```

## Test Patterns

### Spike Test
Sudden increase in traffic.

```javascript
export const options = {
  stages: [
    { duration: '1m', target: 10 },     // Normal traffic
    { duration: '10s', target: 1000 },  // SPIKE!
    { duration: '3m', target: 1000 },   // Stay high
    { duration: '10s', target: 10 },    // Drop
  ],
};
```

### Soak Test
Extended duration at moderate load.

```javascript
export const options = {
  stages: [
    { duration: '5m', target: 100 },   // Ramp up
    { duration: '24h', target: 100 },  // Stay for 24 hours
    { duration: '5m', target: 0 },     // Ramp down
  ],
};
```

### Breakpoint Test
Incrementally increase until system breaks.

```javascript
export const options = {
  stages: [
    { duration: '10m', target: 100 },
    { duration: '10m', target: 200 },
    { duration: '10m', target: 400 },
    { duration: '10m', target: 800 },
    { duration: '10m', target: 1600 },
    { duration: '10m', target: 3200 },
    // Keep going until system fails
  ],
};
```
