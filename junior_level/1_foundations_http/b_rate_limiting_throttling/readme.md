# Rate Limiting & Throttling

Below are **10 build‑style exercises** for Rate Limiting & Throttling in Node.js. Each exercise includes step-by-step instructions and clear acceptance criteria that can be verified with unit or integration tests.

* * *

## 1) Add basic rate limiting to an endpoint

**Goal:** Prevent excessive requests to `/api/ping`.
**Build steps:**

1. Set up an Express app and install `express-rate-limit`.
2. Apply a rate limiter to `/api/ping` (e.g., max 5 requests per minute).
**Acceptance criteria:**

- 5 requests within a minute return 200; 6th returns 429.
- Unit test: Assert status codes and error message.

* * *

## 2) Customize rate limit response

**Goal:** Return a helpful error when limited.
**Build steps:**

1. Configure the limiter to return a JSON error with `Retry-After` header.
2. Include a message in the response body.
**Acceptance criteria:**

- Limited requests return 429, JSON error, and `Retry-After` header.
- Unit test: Assert header and error shape.

* * *

## 3) Apply different limits to different endpoints

**Goal:** Use custom limits for `/api/login` and `/api/register`.
**Build steps:**

1. Set `/api/login` to max 3/min, `/api/register` to max 2/min.
2. Use separate limiter instances for each route.
**Acceptance criteria:**

- Limits enforced per endpoint.
- Unit test: Assert correct limits and error responses.

* * *

## 4) Track rate limits per user/IP

**Goal:** Limit requests per unique user or IP.
**Build steps:**

1. Configure the limiter to use `req.ip` or a user ID from `req.user`.
2. Test with multiple users/IPs.
**Acceptance criteria:**

- Each user/IP has independent limits.
- Integration test: Simulate requests from different users/IPs.

* * *

## 5) Implement global rate limiting

**Goal:** Limit total requests to the API.
**Build steps:**

1. Add a global limiter to all routes (e.g., max 100 requests/minute).
2. Combine with per-route limiters.
**Acceptance criteria:**

- Global limit enforced across all endpoints.
- Integration test: Assert global limit and error response.

* * *

## 6) Add burst and sustained limits

**Goal:** Allow short bursts but limit sustained traffic.
**Build steps:**

1. Configure a burst limit (e.g., 10 requests in 10 seconds) and a sustained limit (e.g., 100/min).
2. Use a library that supports both (e.g., `rate-limiter-flexible`).
**Acceptance criteria:**

- Burst requests allowed, sustained limit enforced.
- Integration test: Simulate burst and sustained traffic.

* * *

## 7) Whitelist trusted users or IPs

**Goal:** Allow unlimited access for certain users/IPs.
**Build steps:**

1. Add a whitelist array for trusted users/IPs.
2. Skip rate limiting for whitelisted entries.
**Acceptance criteria:**

- Whitelisted users/IPs are not limited.
- Unit test: Assert unlimited access for whitelisted entries.

* * *

## 8) Log and monitor rate limit events

**Goal:** Track when limits are hit.
**Build steps:**

1. Add logging for rate limit violations.
2. Integrate with a monitoring tool (e.g., Prometheus, Datadog).
**Acceptance criteria:**

- Violations are logged and visible in monitoring.
- Integration test: Assert logs and monitoring events.

* * *

## 9) Implement throttling for expensive operations

**Goal:** Slow down requests to `/api/export`.
**Build steps:**

1. Add a throttling middleware that delays responses if requests exceed a threshold.
2. Return a 429 if the queue is full.
**Acceptance criteria:**

- Excess requests are delayed or rejected with 429.
- Integration test: Assert delays and error responses.

* * *

## 10) Test rate limiting under concurrent load

**Goal:** Ensure limits work with many simultaneous users.
**Build steps:**

1. Write integration tests that simulate concurrent requests from multiple users/IPs.
2. Assert that limits are enforced and no user can exceed them.
**Acceptance criteria:**

- Limits hold under concurrent load.
- Integration test: Simulate concurrency and verify enforcement.

* * *

### Notes for students

- Test all rate limiting and throttling scenarios.
- Use clear error messages and headers.
- Monitor and log violations for visibility.
- Whitelist trusted users/IPs as needed.
