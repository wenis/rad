# Artillery Load Testing

## Installation
```bash
npm install -g artillery
```

## Configuration File

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

## Running Artillery

**Basic run:**
```bash
artillery run load-test.yml
```

**With HTML report:**
```bash
artillery run load-test.yml --output report.json
artillery report report.json --output report.html
```

## Advanced Features

### Environment Variables

```yaml
config:
  target: '{{ $processEnvironment.API_URL }}'
  phases:
    - duration: 60
      arrivalRate: '{{ $processEnvironment.RATE }}'
```

```bash
API_URL=https://api.example.com RATE=20 artillery run load-test.yml
```

### Custom JavaScript Functions

```yaml
config:
  processor: './helpers.js'

scenarios:
  - flow:
      - function: 'generateRandomUser'
      - post:
          url: '/users'
          json:
            email: '{{ email }}'
            name: '{{ name }}'
```

**helpers.js:**
```javascript
module.exports = {
  generateRandomUser: function(context, events, done) {
    context.vars.email = `user${Date.now()}@example.com`;
    context.vars.name = `User ${Date.now()}`;
    return done();
  }
};
```

## When to Use Artillery

**Pros:**
- Simple YAML configuration
- Easy to read and maintain
- Good for CI/CD pipelines
- Supports AWS Lambda (serverless load testing)
- Built-in scenario support

**Cons:**
- Less mature than k6/JMeter
- Limited advanced features
- JavaScript-based (slower than Go-based tools)
- Community smaller than k6
