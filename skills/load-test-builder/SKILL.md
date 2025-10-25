---
name: load-test-builder
description: Creates load and stress tests using k6, Locust, or Artillery to validate performance under load, identify bottlenecks, and establish baselines. Use before launch OR when scaling OR investigating performance OR capacity planning.
allowed-tools: Read, Write, Bash
---

# Load Test Builder

You create comprehensive load tests that validate system performance, identify bottlenecks, and establish performance baselines.

## When to use
- Before launching to production
- When expecting traffic growth
- After performance optimizations (validate improvements)
- Capacity planning (how much can we handle?)
- Investigating performance issues
- Testing autoscaling behavior
- Stress testing for failure modes

## Load Testing vs Stress Testing

### Load Testing
Test system under expected load to verify it meets performance targets.

**Example**: Can handle 1,000 concurrent users with P95 < 500ms

### Stress Testing
Push system beyond limits to find breaking point.

**Example**: Increase load until errors occur or response time degrades

### Soak Testing
Run at moderate load for extended period.

**Example**: 500 concurrent users for 24 hours (find memory leaks)

## Tool Comparison

| Tool | Language | Pros | Cons |
|------|----------|------|------|
| **k6** | JavaScript | Simple, great UX, Grafana Cloud | No browser/GUI automation |
| **Locust** | Python | Flexible, distributed, web UI | Slower than k6 |
| **Artillery** | JavaScript | Easy, YAML config, serverless | Less mature |
| **JMeter** | Java | Feature-rich, GUI | Heavy, complex |
| **Gatling** | Scala | High performance | Steep learning curve |

**Recommendation**: k6 for API load testing, Playwright for browser testing

## k6 (Recommended)

### Installation
```bash
# macOS
brew install k6

# Linux
sudo gpg -k
sudo gpg --no-default-keyring --keyring /usr/share/keyrings/k6-archive-keyring.gpg --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys C5AD17C747E3415A3642D57D77C6C491D6AC1D69
echo "deb [signed-by=/usr/share/keyrings/k6-archive-keyring.gpg] https://dl.k6.io/deb stable main" | sudo tee /etc/apt/sources.list.d/k6.list
sudo apt-get update
sudo apt-get install k6

# Docker
docker pull grafana/k6
```

### Basic Load Test

```javascript
// load-test.js
import http from 'k6/http';
import { check, sleep } from 'k6';

// Test configuration
export const options = {
  // Ramp up to 100 virtual users over 2 minutes
  stages: [
    { duration: '2m', target: 100 },  // Ramp up
    { duration: '5m', target: 100 },  // Stay at 100
    { duration: '2m', target: 0 },    // Ramp down
  ],

  // Performance thresholds
  thresholds: {
    http_req_duration: ['p(95)<500'], // 95% of requests < 500ms
    http_req_failed: ['rate<0.01'],   // Error rate < 1%
  },
};

export default function () {
  // GET request
  const res = http.get('https://api.example.com/users');

  // Assertions
  check(res, {
    'status is 200': (r) => r.status === 200,
    'response time < 500ms': (r) => r.timings.duration < 500,
    'has users': (r) => JSON.parse(r.body).length > 0,
  });

  sleep(1); // Wait 1 second between iterations
}
```

**Run:**
```bash
k6 run load-test.js

# With output to InfluxDB + Grafana
k6 run --out influxdb=http://localhost:8086/k6 load-test.js

# Generate HTML report
k6 run --out json=results.json load-test.js
k6-html-reporter results.json --output report.html
```

### Advanced k6 Examples

**POST with authentication:**
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

**Multiple endpoints (realistic scenario):**
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

**Stress test (find breaking point):**
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

## Locust (Python)

### Installation
```bash
pip install locust
```

### Basic Load Test

```python
# locustfile.py
from locust import HttpUser, task, between

class WebsiteUser(HttpUser):
    wait_time = between(1, 3)  # Wait 1-3 seconds between tasks

    def on_start(self):
        # Login once when user starts
        response = self.client.post("/auth/login", json={
            "email": "test@example.com",
            "password": "password123"
        })
        self.token = response.json()["access_token"]

    @task(3)  # Weight: 3x more likely than create
    def list_users(self):
        self.client.get("/users", headers={
            "Authorization": f"Bearer {self.token}"
        })

    @task(2)
    def get_user(self):
        user_id = random.randint(1, 1000)
        self.client.get(f"/users/{user_id}", headers={
            "Authorization": f"Bearer {self.token}"
        })

    @task(1)
    def create_user(self):
        self.client.post("/users", json={
            "email": f"user{time.time()}@example.com",
            "name": f"User {time.time()}"
        }, headers={
            "Authorization": f"Bearer {self.token}"
        })
```

**Run:**
```bash
# Web UI (http://localhost:8089)
locust -f locustfile.py --host=https://api.example.com

# Headless (command line)
locust -f locustfile.py --host=https://api.example.com \
  --users 100 --spawn-rate 10 --run-time 10m --headless
```

### Distributed Load Testing

**Master:**
```bash
locust -f locustfile.py --master --expect-workers 4
```

**Workers (run on multiple machines):**
```bash
locust -f locustfile.py --worker --master-host=MASTER_IP
```

## Artillery

### Installation
```bash
npm install -g artillery
```

### Configuration File

```yaml
# load-test.yml
config:
  target: 'https://api.example.com'
  phases:
    - duration: 60
      arrivalRate: 10  # 10 users per second
    - duration: 120
      arrivalRate: 20  # Ramp to 20 users/sec
    - duration: 60
      arrivalRate: 10  # Ramp down

  defaults:
    headers:
      Content-Type: 'application/json'

scenarios:
  - name: 'User Flow'
    flow:
      # Login
      - post:
          url: '/auth/login'
          json:
            email: 'test@example.com'
            password: 'password123'
          capture:
            - json: '$.access_token'
              as: 'token'

      # List users
      - get:
          url: '/users'
          headers:
            Authorization: 'Bearer {{ token }}'

      # Get specific user
      - get:
          url: '/users/{{ $randomNumber(1, 1000) }}'
          headers:
            Authorization: 'Bearer {{ token }}'

      # Create user
      - post:
          url: '/users'
          headers:
            Authorization: 'Bearer {{ token }}'
          json:
            email: 'user{{ $randomString(10) }}@example.com'
            name: 'User {{ $randomString(10) }}'

      - think: 2  # Pause 2 seconds
```

**Run:**
```bash
artillery run load-test.yml

# With HTML report
artillery run load-test.yml --output report.json
artillery report report.json --output report.html
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

## Metrics to Track

### Response Time
- **P50 (median)**: 50% of requests faster
- **P95**: 95% of requests faster (target SLA)
- **P99**: 99% of requests faster
- **Max**: Worst case

### Error Rate
- **Total errors**: Count
- **Error percentage**: Rate
- **Error types**: 4xx vs 5xx

### Throughput
- **Requests per second**: Total load
- **Successful requests per second**: Actual capacity

### Resource Usage
- **CPU**: Should not max out
- **Memory**: Watch for leaks
- **Database connections**: Monitor pool
- **Network bandwidth**: Check saturation

## Results Analysis

**k6 summary:**
```
checks.........................: 99.50% ✓ 9950  ✗ 50
data_received..................: 12 MB  40 kB/s
data_sent......................: 1.2 MB 4.0 kB/s
http_req_blocked...............: avg=1.2ms  min=0s  med=1ms   max=100ms p(95)=3ms
http_req_duration..............: avg=247ms  min=50ms med=200ms max=2s    p(95)=450ms
http_req_failed................: 0.50%  ✓ 50    ✗ 9950
http_reqs......................: 10000  33.33/s
vus............................: 100    min=0   max=100
vus_max........................: 100    min=100 max=100
```

**Interpretation:**
- ✅ 99.5% success rate (good)
- ✅ P95 = 450ms (under 500ms threshold)
- ⚠️ Max = 2s (some requests very slow)
- 🔍 Investigate: Why 50 requests failed?

## CI/CD Integration

```yaml
# .github/workflows/load-test.yml
name: Load Test

on:
  pull_request:
    branches: [main]

jobs:
  load-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Run k6 load test
        uses: grafana/k6-action@v0.3.0
        with:
          filename: tests/load-test.js
          cloud: true
          token: ${{ secrets.K6_CLOUD_TOKEN }}

      - name: Check thresholds
        run: |
          if grep -q "✗" k6-results.txt; then
            echo "Load test failed - performance thresholds not met"
            exit 1
          fi
```

## Best Practices

✅ **DO:**
- Test in staging first (not production!)
- Ramp up gradually (don't spike immediately)
- Use realistic data and scenarios
- Monitor server resources during test
- Run multiple iterations (results vary)
- Test from multiple regions
- Clean up test data after

❌ **DON'T:**
- Test production without permission
- Ignore server-side metrics
- Only test happy path
- Hardcode credentials in test scripts
- Run tests during peak hours
- Test with admin accounts
- Leave test data in production

## Reporting

```markdown
# Load Test Report - API v2.0

**Date**: 2025-01-24
**Duration**: 20 minutes
**Tool**: k6
**Target**: https://api-staging.example.com

## Test Configuration

- **Virtual Users**: 100 (ramped over 2 min)
- **Duration**: 5 minutes at peak
- **Scenario**: Mixed read/write operations
- **Ramp Down**: 2 minutes

## Results Summary

| Metric | Target | Actual | Status |
|--------|--------|--------|--------|
| P95 Response Time | < 500ms | 420ms | ✅ Pass |
| Error Rate | < 1% | 0.3% | ✅ Pass |
| Throughput | > 50 req/s | 67 req/s | ✅ Pass |

## Detailed Metrics

- **Total Requests**: 20,100
- **Successful**: 20,040 (99.7%)
- **Failed**: 60 (0.3%)
- **Data Transferred**: 45 MB
- **Average Response**: 247ms
- **P95 Response**: 420ms
- **P99 Response**: 680ms
- **Max Response**: 1.8s

## Errors

- **500 Internal Server Error**: 45 (timeout on /users/{id}/posts)
- **429 Too Many Requests**: 15 (rate limit on /search)

## Server Resources (during peak)

- **CPU Usage**: 65% (max 80%)
- **Memory**: 4.2 GB / 8 GB (52%)
- **Database Connections**: 45 / 100 (45%)
- **Network**: 12 Mbps in, 8 Mbps out

## Bottlenecks Identified

1. **Database query on /users/{id}/posts**
   - N+1 query problem
   - Recommendation: Add DataLoader

2. **Rate limiting too aggressive on /search**
   - Currently 10 req/min
   - Recommendation: Increase to 30 req/min

## Recommendations

1. Optimize /users/{id}/posts endpoint (add eager loading)
2. Increase rate limit on /search
3. Add caching for popular queries
4. Monitor connection pool usage (approaching limit)

## Next Steps

- Fix identified bottlenecks
- Re-run load test
- Load test with 200 VUs (2x current)
```

## Instructions

1. **Define objectives**: What are you testing? What's the SLA?
2. **Choose tool**: k6 for APIs, Playwright for browser
3. **Create realistic scenarios**: Match production usage patterns
4. **Set thresholds**: P95 < 500ms, error rate < 1%, etc.
5. **Run in staging**: Never production without explicit approval
6. **Analyze results**: Look for bottlenecks, errors, resource usage
7. **Generate report**: Document findings and recommendations
8. **Fix issues**: Address bottlenecks
9. **Repeat**: Validate improvements

## Constraints

- Must not run against production without approval
- Must clean up test data
- Must monitor server resources
- Must set realistic thresholds
- Results must be reproducible
- Must test from staging/similar environment
