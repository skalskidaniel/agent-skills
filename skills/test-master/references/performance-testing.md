# Performance Testing

## Locust Load Test

```python
from locust import HttpUser, LoadTestShape, between, events, task

response_times_ms = []
failures = 0
requests = 0


@events.request.add_listener
def _record(response_time, exception, **kwargs):
    global failures, requests
    requests += 1
    response_times_ms.append(response_time)
    if exception:
        failures += 1


@events.quitting.add_listener
def _check_thresholds(environment, **kwargs):
    # 95% of requests under 500ms, and fewer than 1% errors
    if not response_times_ms:
        environment.process_exit_code = 1
        return
    ordered = sorted(response_times_ms)
    p95 = ordered[int(0.95 * (len(ordered) - 1))]
    error_rate = failures / requests
    if p95 >= 500 or error_rate >= 0.01:
        environment.process_exit_code = 1


class WebsiteUser(HttpUser):
    wait_time = between(1, 1)

    @task
    def get_users(self):
        with self.client.get("/api/users", catch_response=True) as response:
            if response.status_code != 200:
                response.failure("status is not 200")
            elif response.elapsed.total_seconds() >= 0.2:
                response.failure("response time >= 200ms")


class LoadShape(LoadTestShape):
    # (stage length in seconds, target users)
    stages = [
        (30, 20),  # Ramp up to 20 users
        (60, 20),  # Stay at 20 users
        (30, 0),   # Ramp down
    ]

    def tick(self):
        run_time = self.get_run_time()
        elapsed = 0
        for duration, users in self.stages:
            elapsed += duration
            if run_time < elapsed:
                return users, max(users, 1)
        return None
```

Run against the app with `locust -f locustfile.py --headless --host http://localhost:3000`.

## Stress Test

```python
from locust import LoadTestShape


class StressShape(LoadTestShape):
    stages = [
        (120, 100),  # Ramp to 100 users
        (300, 100),  # Stay at 100
        (120, 200),  # Push to 200
        (300, 200),  # Stay at 200
        (120, 0),    # Ramp down
    ]

    def tick(self):
        run_time = self.get_run_time()
        elapsed = 0
        for duration, users in self.stages:
            elapsed += duration
            if run_time < elapsed:
                return users, max(users, 1)
        return None
```

## Spike Test

```python
from locust import LoadTestShape


class SpikeShape(LoadTestShape):
    stages = [
        (10, 10),    # Normal load
        (60, 10),
        (10, 200),   # Spike
        (180, 200),
        (10, 10),    # Scale down
        (180, 10),
        (10, 0),
    ]

    def tick(self):
        run_time = self.get_run_time()
        elapsed = 0
        for duration, users in self.stages:
            elapsed += duration
            if run_time < elapsed:
                return users, max(users, 1)
        return None
```

## API Testing with Auth

```python
from locust import HttpUser, task


class AuthenticatedUser(HttpUser):
    def on_start(self):
        response = self.client.post("/api/login", json={
            "email": "test@test.com",
            "password": "password",
        })
        token = response.json()["token"]
        self.client.headers["Authorization"] = f"Bearer {token}"

    @task
    def get_protected(self):
        self.client.get("/api/protected")
```

## Thresholds Reference

```python
def check_thresholds(samples_ms, failures, requests, login_samples_ms, run_duration_seconds):
    ordered = sorted(samples_ms)
    p95 = ordered[int(0.95 * (len(ordered) - 1))]
    p99 = ordered[int(0.99 * (len(ordered) - 1))]
    error_rate = failures / requests
    throughput = requests / run_duration_seconds

    login_ordered = sorted(login_samples_ms)
    login_p95 = login_ordered[int(0.95 * (len(login_ordered) - 1))]

    assert p95 < 500 and p99 < 1000
    assert error_rate < 0.01
    assert throughput > 100
    assert login_p95 < 200
```

## Quick Reference

| Metric               | Description                         |
| -------------------- | ----------------------------------- |
| `response.elapsed`   | Response time                       |
| `response.failure()` | Count a request as failed           |
| Requests per second  | Throughput                          |
| 95th percentile      | Sort samples and read the 95% index |
| `wait_time`          | Pause between tasks                 |

| Test Type | Purpose                 |
| --------- | ----------------------- |
| Load      | Normal expected load    |
| Stress    | Find breaking point     |
| Spike     | Sudden traffic surge    |
| Soak      | Long duration stability |
