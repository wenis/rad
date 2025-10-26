# Advanced Error Handling Patterns

Advanced patterns for resilient error handling.

## Retry Logic

Automatic retry for transient failures with exponential backoff.

```typescript
// utils/retry.ts
export async function retry<T>(
  fn: () => Promise<T>,
  options: {
    maxAttempts?: number;
    delay?: number;
    backoff?: 'exponential' | 'linear';
    shouldRetry?: (error: Error) => boolean;
  } = {}
): Promise<T> {
  const {
    maxAttempts = 3,
    delay = 1000,
    backoff = 'exponential',
    shouldRetry = () => true,
  } = options;

  let lastError: Error;

  for (let attempt = 1; attempt <= maxAttempts; attempt++) {
    try {
      return await fn();
    } catch (error) {
      lastError = error as Error;

      if (attempt === maxAttempts || !shouldRetry(lastError)) {
        throw lastError;
      }

      const waitTime = backoff === 'exponential'
        ? delay * Math.pow(2, attempt - 1)
        : delay * attempt;

      await new Promise(resolve => setTimeout(resolve, waitTime));
    }
  }

  throw lastError!;
}

// Usage
const user = await retry(
  () => apiClient.getUser(id),
  {
    maxAttempts: 3,
    delay: 1000,
    backoff: 'exponential',
    shouldRetry: (error) => {
      // Retry on network errors, not on 404
      return error instanceof ExternalServiceError;
    },
  }
);
```

### Python Implementation

```python
import asyncio
from typing import TypeVar, Callable, Optional
import logging

T = TypeVar('T')
logger = logging.getLogger(__name__)

async def retry(
    fn: Callable[[], T],
    max_attempts: int = 3,
    delay: float = 1.0,
    backoff: str = 'exponential',
    should_retry: Optional[Callable[[Exception], bool]] = None
) -> T:
    """Retry a function with exponential backoff"""
    if should_retry is None:
        should_retry = lambda _: True

    last_error = None

    for attempt in range(1, max_attempts + 1):
        try:
            return await fn()
        except Exception as error:
            last_error = error

            if attempt == max_attempts or not should_retry(error):
                raise last_error

            wait_time = (
                delay * (2 ** (attempt - 1))
                if backoff == 'exponential'
                else delay * attempt
            )

            logger.warning(
                f"Attempt {attempt} failed, retrying in {wait_time}s",
                extra={"error": str(error), "attempt": attempt}
            )

            await asyncio.sleep(wait_time)

    raise last_error

# Usage
user = await retry(
    lambda: api_client.get_user(user_id),
    max_attempts=3,
    delay=1.0,
    backoff='exponential',
    should_retry=lambda e: isinstance(e, ExternalServiceError)
)
```

## Circuit Breaker

Prevent cascading failures by "opening" the circuit after threshold failures.

```typescript
// utils/circuitBreaker.ts
type CircuitState = 'CLOSED' | 'OPEN' | 'HALF_OPEN';

export class CircuitBreaker {
  private state: CircuitState = 'CLOSED';
  private failures = 0;
  private nextAttempt = Date.now();

  constructor(
    private threshold: number = 5,
    private timeout: number = 60000
  ) {}

  async execute<T>(fn: () => Promise<T>): Promise<T> {
    if (this.state === 'OPEN') {
      if (Date.now() < this.nextAttempt) {
        throw new Error('Circuit breaker is OPEN');
      }
      this.state = 'HALF_OPEN';
    }

    try {
      const result = await fn();
      this.onSuccess();
      return result;
    } catch (error) {
      this.onFailure();
      throw error;
    }
  }

  private onSuccess() {
    this.failures = 0;
    this.state = 'CLOSED';
  }

  private onFailure() {
    this.failures++;
    if (this.failures >= this.threshold) {
      this.state = 'OPEN';
      this.nextAttempt = Date.now() + this.timeout;
    }
  }

  getState(): CircuitState {
    return this.state;
  }
}

// Usage
const breaker = new CircuitBreaker(5, 60000);

try {
  const data = await breaker.execute(() => externalAPI.getData());
} catch (error) {
  // Circuit is open or request failed
  logger.error('Circuit breaker tripped', { state: breaker.getState() });
}
```

### Python Implementation

```python
from enum import Enum
from typing import TypeVar, Callable
import time
import logging

T = TypeVar('T')
logger = logging.getLogger(__name__)

class CircuitState(Enum):
    CLOSED = "CLOSED"
    OPEN = "OPEN"
    HALF_OPEN = "HALF_OPEN"

class CircuitBreaker:
    def __init__(self, threshold: int = 5, timeout: float = 60.0):
        self.threshold = threshold
        self.timeout = timeout
        self.state = CircuitState.CLOSED
        self.failures = 0
        self.next_attempt = time.time()

    async def execute(self, fn: Callable[[], T]) -> T:
        if self.state == CircuitState.OPEN:
            if time.time() < self.next_attempt:
                raise Exception("Circuit breaker is OPEN")
            self.state = CircuitState.HALF_OPEN

        try:
            result = await fn()
            self._on_success()
            return result
        except Exception as error:
            self._on_failure()
            raise error

    def _on_success(self):
        self.failures = 0
        self.state = CircuitState.CLOSED

    def _on_failure(self):
        self.failures += 1
        if self.failures >= self.threshold:
            self.state = CircuitState.OPEN
            self.next_attempt = time.time() + self.timeout
            logger.warning(f"Circuit breaker opened after {self.failures} failures")

    def get_state(self) -> CircuitState:
        return self.state

# Usage
breaker = CircuitBreaker(threshold=5, timeout=60.0)

try:
    data = await breaker.execute(lambda: external_api.get_data())
except Exception as error:
    logger.error(f"Circuit breaker error: {error}", extra={"state": breaker.get_state().value})
```

## When to Use These Patterns

### Retry
- Network requests to external services
- Database queries (transient failures)
- File operations (temporary locks)

**Don't retry:**
- Validation errors (400)
- Not found errors (404)
- Authorization errors (401, 403)

### Circuit Breaker
- Calls to external services
- Database connections
- Third-party APIs

**Benefits:**
- Prevents cascade failures
- Allows failing service time to recover
- Improves system resilience
