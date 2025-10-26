# Locust (Python) Load Testing

## Installation
```bash
pip install locust
```

## Basic Load Test

```python
# locustfile.py
from locust import HttpUser, task, between
import random
import time

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

## Running Locust

**Web UI (http://localhost:8089):**
```bash
locust -f locustfile.py --host=https://api.example.com
```

**Headless (command line):**
```bash
locust -f locustfile.py --host=https://api.example.com \
  --users 100 --spawn-rate 10 --run-time 10m --headless
```

## Distributed Load Testing

**Master:**
```bash
locust -f locustfile.py --master --expect-workers 4
```

**Workers (run on multiple machines):**
```bash
locust -f locustfile.py --worker --master-host=MASTER_IP
```

## Advanced Patterns

### Custom Load Shape

```python
from locust import LoadTestShape

class StagesShape(LoadTestShape):
    """
    A simple load test shape class that mimics k6 stages
    """
    stages = [
        {"duration": 120, "users": 10, "spawn_rate": 10},
        {"duration": 300, "users": 50, "spawn_rate": 10},
        {"duration": 120, "users": 100, "spawn_rate": 20},
        {"duration": 300, "users": 100, "spawn_rate": 20},
        {"duration": 120, "users": 0, "spawn_rate": 20},
    ]

    def tick(self):
        run_time = self.get_run_time()

        for stage in self.stages:
            if run_time < stage["duration"]:
                tick_data = (stage["users"], stage["spawn_rate"])
                return tick_data

        return None
```

### Event Hooks

```python
from locust import events

@events.test_start.add_listener
def on_test_start(environment, **kwargs):
    print("Test starting!")

@events.test_stop.add_listener
def on_test_stop(environment, **kwargs):
    print("Test stopped!")
```

## When to Use Locust

**Pros:**
- Python-based (familiar for Python developers)
- Flexible and programmable
- Great web UI for monitoring
- Distributed testing support
- Good for complex user flows

**Cons:**
- Slower than k6 (GIL limitations)
- More verbose configuration
- Requires Python knowledge
