# Advanced Error Handling & Resilience

Below are **10 build‑style exercises** for Advanced Error Handling & Resilience in Node.js. Each exercise includes step-by-step instructions and clear acceptance criteria that can be verified with unit or integration tests.

* * *

## 1) Implement centralized error middleware

**Goal:** Standardize error responses across the app.
**Build steps:**

1. Add an Express error handler `(err, req, res, next)`.
2. Return JSON `{ error: { message, code } }` for all errors.
**Acceptance criteria:**

- All errors are caught and returned in a consistent format.
- Unit test: Assert error handler catches and formats errors.

* * *

## 2) Handle async errors and promise rejections

**Goal:** Catch errors from async routes and promises.
**Build steps:**

1. Use a wrapper for async route handlers to catch errors.
2. Listen for `unhandledRejection` and log the error.
**Acceptance criteria:**

- Async errors are handled and logged.
- Unit test: Assert async errors are caught and logged.

* * *

## 3) Map known errors to proper status codes

**Goal:** Return correct HTTP status for different error types.
**Build steps:**

1. Define custom error classes (e.g., `NotFoundError`, `ValidationError`).
2. Map errors to status codes in the error handler.
**Acceptance criteria:**

- Known errors return mapped status codes; unknown errors return 500.
- Unit test: Assert status codes for different error types.

* * *

## 4) Log errors with request context

**Goal:** Include request info in error logs.
**Build steps:**

1. Log errors with method, path, and request ID.
2. Use a logger like `pino` or `winston`.
**Acceptance criteria:**

- Error logs include request context.
- Unit test: Assert log entries contain context.

* * *

## 5) Implement graceful degradation for failed dependencies

**Goal:** Return fallback responses when dependencies fail.
**Build steps:**

1. Detect when a third-party API or DB is unavailable.
2. Return a fallback response or cached data.
**Acceptance criteria:**

- Fallback is returned when dependency fails.
- Integration test: Simulate failure and assert fallback.

* * *

## 6) Add retry logic for transient errors

**Goal:** Retry failed operations when appropriate.
**Build steps:**

1. Use a library like `p-retry` for retrying failed requests.
2. Limit retries and log each attempt.
**Acceptance criteria:**

- Transient errors are retried up to the limit.
- Unit test: Assert retry behavior and logging.

* * *

## 7) Use circuit breakers for unstable services

**Goal:** Prevent repeated failures from overwhelming the app.
**Build steps:**

1. Integrate a circuit breaker (e.g., `opossum`) for external calls.
2. Open the breaker after repeated failures; close after recovery.
**Acceptance criteria:**

- Circuit breaker opens/closes as expected.
- Integration test: Simulate failures and assert breaker state.

* * *

## 8) Return helpful error messages to clients

**Goal:** Make error responses clear and actionable.
**Build steps:**

1. Include error code, message, and possible resolution in responses.
2. Avoid exposing sensitive info in error messages.
**Acceptance criteria:**

- Error responses are clear and do not leak sensitive data.
- Unit test: Assert error message content.

* * *

## 9) Test error handling with integration tests

**Goal:** Ensure error handling works end-to-end.
**Build steps:**

1. Write integration tests for common error scenarios (404, 400, 500).
2. Assert correct status codes and response shapes.
**Acceptance criteria:**

- Integration tests pass for all error cases.
- Integration test: Assert status and response for each error.

* * *

## 10) Monitor and alert on error rates

**Goal:** Track and respond to high error rates in production.
**Build steps:**

1. Integrate error metrics with a monitoring tool (e.g., Prometheus).
2. Set up alerts for error spikes or critical failures.
**Acceptance criteria:**

- Error metrics are tracked and alerts are triggered on spikes.
- Integration test: Simulate error spike and assert alerting.

* * *

### Notes for students

- Always test error handling for all endpoints.
- Use clear, consistent error responses and logging.
- Monitor error rates and set up alerts for reliability.
