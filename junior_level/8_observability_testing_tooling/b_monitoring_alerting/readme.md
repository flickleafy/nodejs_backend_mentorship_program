# Monitoring & Alerting

Below are **10 build‑style exercises** for Monitoring & Alerting in Node.js. Each exercise includes step-by-step instructions and clear acceptance criteria that can be verified with unit or integration tests.

* * *

## 1) Add basic request logging

**Goal:** Log every incoming request.
**Build steps:**

1. Use a logger (e.g., `pino`, `winston`) to log method, path, and status.
2. Log to console or file.
**Acceptance criteria:**

- All requests are logged with method, path, and status.
- Unit test: Assert log entries for requests.

* * *

## 2) Log errors and exceptions

**Goal:** Track errors in logs.
**Build steps:**

1. Log all errors caught by error middleware.
2. Include stack trace and request context.
**Acceptance criteria:**

- Errors are logged with context and stack trace.
- Unit test: Assert error logs for simulated errors.

* * *

## 3) Expose application metrics endpoint

**Goal:** Provide metrics for monitoring tools.
**Build steps:**

1. Use `prom-client` to collect metrics.
2. Expose `GET /metrics` endpoint.
**Acceptance criteria:**

- `/metrics` returns standard and custom metrics.
- Integration test: Assert metrics endpoint and content.

* * *

## 4) Track business metrics (counters & gauges)

**Goal:** Measure domain-specific activity.
**Build steps:**

1. Add counters for key actions (e.g., tasks created).
2. Add gauges for current state (e.g., active sessions).
**Acceptance criteria:**

- Metrics are updated on relevant actions.
- Unit test: Assert metric values after actions.

* * *

## 5) Monitor slow requests and latency

**Goal:** Track request duration and flag slow requests.
**Build steps:**

1. Measure request duration in middleware.
2. Log or increment a metric for requests >500ms.
**Acceptance criteria:**

- Slow requests are tracked and logged.
- Unit test: Assert detection and logging of slow requests.

* * *

## 6) Integrate with external monitoring tools

**Goal:** Send metrics and logs to a service (e.g., Datadog, Prometheus).
**Build steps:**

1. Configure integration with chosen tool.
2. Verify data is received and displayed in dashboard.
**Acceptance criteria:**

- Metrics/logs appear in external dashboard.
- Integration test: Assert data transmission and visibility.

* * *

## 7) Set up automated alerts for errors

**Goal:** Notify on critical errors or downtime.
**Build steps:**

1. Configure alerting in monitoring tool for error spikes or service downtime.
2. Test alert triggers.
**Acceptance criteria:**

- Alerts are triggered on error/downtime events.
- Integration test: Simulate error/downtime and assert alerting.

* * *

## 8) Monitor resource usage (CPU, memory)

**Goal:** Track system health.
**Build steps:**

1. Use Node.js or monitoring tool to collect CPU and memory usage.
2. Expose or log resource metrics.
**Acceptance criteria:**

- Resource metrics are collected and accessible.
- Unit test: Assert resource metrics are present.

* * *

## 9) Visualize metrics with dashboards

**Goal:** Create dashboards for key metrics.
**Build steps:**

1. Use Grafana, Datadog, or similar to visualize metrics.
2. Add charts for requests, errors, latency, and business metrics.
**Acceptance criteria:**

- Dashboards display real-time metrics.
- Integration test: Assert dashboard setup and data accuracy.

* * *

## 10) Test monitoring and alerting under load

**Goal:** Ensure monitoring and alerting work at scale.
**Build steps:**

1. Simulate high traffic and error rates.
2. Verify metrics, logs, and alerts are accurate and timely.
**Acceptance criteria:**

- Monitoring and alerting work under load.
- Integration test: Simulate load and assert monitoring/alerting.

* * *

### Notes for students

- Always log requests and errors.
- Track key metrics and visualize them.
- Set up alerts for reliability and fast response.
- Test monitoring and alerting under real-world conditions.
