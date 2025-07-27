# Observability, Testing, and Tooling

Below are **20 “Observability, Testing, and Tooling” build‑style exercises** for a Node.js (JavaScript) backend. They progress from junior to near‑mid level. Prefer **Express** and keep responses consistent (e.g., `{ data, meta, error }`). Suggested libs: logging (**pino**, **pino-http**), metrics (**prom-client**), tracing (**OpenTelemetry**), tests (**Jest**, **supertest**, **nock**, **testcontainers**), lint/format (**ESLint**, **Prettier**), hooks (**husky**, **lint-staged**, **commitlint**), mutation (**Stryker**), load tests (**autocannon**).

Each item includes **step‑by‑step rubrics** and **acceptance criteria**.

* * *

## 1) Structured logging with request/trace correlation

**Goal:** Emit structured JSON logs and correlate a request across lines.  
**Build steps:**

1. Add `pino` + `pino-http` middleware to inject a per‑request `requestId` (UUID) and `traceId` field on `req.log`.

2. Ensure all logs go through `req.log` so the IDs appear on every line.

3. Redact secrets (`Authorization`, `Set-Cookie`, `password`) via `redact` option.  
    **Acceptance criteria:**

- Every request produces at least one log line with `requestId` and `traceId`.

- Secret fields are replaced by `[Redacted]`.

- Log levels switchable via `LOG_LEVEL` env.

* * *

## 2) Dev vs prod logging transports & sampling

**Goal:** Keep dev readable, prod efficient.  
**Build steps:**

1. In dev, attach a pretty transport (pino‑pretty); in prod, write NDJSON to stdout.

2. Add **sampling** for successful 2xx logs (e.g., log only 1/10) while always logging 4xx/5xx.

3. Add a “slow request” log when latency > 500 ms.  
    **Acceptance criteria:**

- Dev shows colored, human‑friendly logs; prod shows compact JSON.

- Sampling reduces info logs but never drops errors.

- Slow requests include `durationMs` and route.

* * *

## 3) HTTP server metrics with `prom-client`

**Goal:** Expose standard and custom metrics.  
**Build steps:**

1. Register default process metrics (`collectDefaultMetrics`).

2. Create a `Histogram` for HTTP durations labeled by `{method, route, status}` (normalize route names).

3. Expose `GET /metrics` (text format).  
    **Acceptance criteria:**

- `/metrics` contains defaults plus `http_request_duration_seconds_bucket`.

- Labels show normalized routes (`/tasks/:id`, not `/tasks/abc`).

* * *

## 4) Business metrics (counters & gauges)

**Goal:** Measure domain activity.  
**Build steps:**

1. Add a `Counter` `tasks_created_total` and increment on successful `POST /tasks`.

2. Add a `Gauge` `queue_depth` (if using a queue) or `active_sessions`.

3. Document all metric names and units.  
    **Acceptance criteria:**

- Creating tasks increases the counter.

- Gauge reflects current values and resets correctly on restart (if appropriate).

* * *

## 5) Basic OpenTelemetry tracing (HTTP + DB)

**Goal:** Generate traces for requests and DB calls.  
**Build steps:**

1. Install OTel SDK, set up `@opentelemetry/auto-instrumentations-node` (http, express, pg/knex).

2. Export to console or OTLP (localhost).

3. Add manual spans around custom logic (e.g., cache lookup).  
    **Acceptance criteria:**

- Trace for a request contains child spans for Express and DB query.

- Span attributes include `http.method`, `db.system`, `db.statement` (sanitized).

* * *

## 6) Trace/log correlation

**Goal:** Make logs searchable by trace.  
**Build steps:**

1. On each request, if a W3C `traceparent` header exists, use its `traceId`; otherwise create one and set `traceparent` on the response.

2. Inject the same `traceId` into pino log bindings.  
    **Acceptance criteria:**

- Requests with inbound `traceparent` keep the same `traceId`.

- Logs for a request share the trace ID visible in traces.

* * *

## 7) Health, readiness, and /diag endpoints

**Goal:** Operational visibility and safe rollouts.  
**Build steps:**

1. `/livez`: always 200 if event loop responsive.

2. `/readyz`: verify DB, Redis, migrations; return 503 if any check fails (list failing checks).

3. `/diagz`: return build info (commit, version), uptime, memory, and selected **non‑secret** config flags.  
    **Acceptance criteria:**

- Breaking a dependency flips `/readyz` to 503 with a specific reason.

- `/diagz` shows commit SHA and uptime; no secrets appear.

* * *

## 8) Central error handler + error metrics

**Goal:** Consistent errors with observability.  
**Build steps:**

1. Centralize error handling to emit JSON `{ error: { code, message } }`.

2. Increment a `Counter` for error occurrences labeled by `code` and `status`.

3. Add logging of stack traces at `error` level (not in response).  
    **Acceptance criteria:**

- Known errors map to stable `code`s and labels.

- `/metrics` shows error counters growing when you hit failure paths.

* * *

## 9) Unit tests (Jest) for controllers/services

**Goal:** Fast, isolated tests.  
**Build steps:**

1. Configure Jest; mock repositories and external clients.

2. Test happy path and edge cases for `tasks` controller: status codes, bodies, validations.

3. Enforce coverage thresholds (e.g., 80% lines/branches).  
    **Acceptance criteria:**

- Running `npm test` passes with coverage ≥ thresholds.

- Controller tests do not touch the real DB.

* * *

## 10) Integration tests with `supertest`

**Goal:** Verify the API surface end‑to‑end (app only).  
**Build steps:**

1. Boot the Express app against an **ephemeral** test DB (sqlite in‑memory or a temp schema).

2. Seed minimal data; test `/auth` + `/tasks` happy and error flows.

3. Isolate tests (truncate tables or wrap each test in a transaction and roll back).  
    **Acceptance criteria:**

- API paths return expected codes and shapes.

- Tests are order‑independent and leave no residue.

* * *

## 11) External HTTP mocking with `nock`

**Goal:** Deterministic integration tests for outbound calls.  
**Build steps:**

1. Wrap all outbound HTTP via a small client module.

2. Use `nock` to stub responses for success, 5xx, timeouts.

3. Assert retries, backoffs, and error mapping.  
    **Acceptance criteria:**

- No real network calls during tests.

- Retry behavior verified by `nock` call counters.

* * *

## 12) Testcontainers for Postgres/Redis

**Goal:** Realistic dependencies in tests.  
**Build steps:**

1. Use `testcontainers` to spin up Postgres and Redis per test suite.

2. Run migrations/seeds; inject container URLs into the app.

3. Clean up containers after tests.  
    **Acceptance criteria:**

- Tests pass on clean machines (CI) with containers.

- DB/Redis connectivity is real (no mocks), and isolation is preserved.

* * *

## 13) Snapshot tests for logs and metrics

**Goal:** Guard log structure and metric names.  
**Build steps:**

1. Capture one request’s log output with a custom stream and snapshot it (stable fields only).

2. Scrape `/metrics` and snapshot the **presence** of metric names and labels (not raw values).  
    **Acceptance criteria:**

- Changing a log key name or metric label breaks the snapshot, prompting a review.

* * *

## 14) Mutation testing with Stryker

**Goal:** Measure test suite strength.  
**Build steps:**

1. Configure Stryker to mutate controllers/services (exclude infra).

2. Run mutation tests in CI (nightly or on demand).

3. Set a baseline mutation score (e.g., ≥ 60%) and track trends.  
    **Acceptance criteria:**

- A surviving mutant indicates missing tests; add tests until score meets baseline.

* * *

## 15) Linting & formatting gates (ESLint + Prettier)

**Goal:** Enforce code style and catch bugs early.  
**Build steps:**

1. Configure ESLint (Google JS Style Guide) + `eslint-plugin-security` and `eslint-plugin-promise`.

2. Add Prettier and resolve rule conflicts with `eslint-config-prettier`.

3. `npm run lint` and `npm run format:check` must pass in CI.  
    **Acceptance criteria:**

- Lint violations fail CI; autofix cleans most issues.

- No style drift after Prettier.

* * *

## 16) Pre‑commit hooks & conventional commits

**Goal:** Automate quality checks on commit.  
**Build steps:**

1. Add `husky` + `lint-staged` to run ESLint/Prettier on staged files.

2. Add `commitlint` with “conventional‑commits” config; reject invalid messages.

3. Optional: `semantic‑release` for automated versioning/changelog on `main`.  
    **Acceptance criteria:**

- Bad commit messages are rejected locally.

- Only formatted, lint‑clean code can be committed.

- Releases cut automatically with changelog (if enabled).

* * *

## 17) CI pipeline (matrix, caching, artifacts)

**Goal:** Reproducible builds and fast feedback.  
**Build steps:**

1. Create GitHub Actions (or similar) with Node version matrix (e.g., 18, 20).

2. Cache `~/.npm` and test container layers; run lint → unit → integration → mutation (possibly nightly).

3. Upload coverage to a service and store junit/coverage artifacts.  
    **Acceptance criteria:**

- CI runs in parallel and finishes under a reasonable time budget.

- A failing stage blocks later deploy jobs.

* * *

## 18) Source‑maps & actionable stack traces

**Goal:** Debuggable errors in prod.  
**Build steps:**

1. Enable `source-map-support` and produce source maps for compiled/bundled code (if bundling).

2. Ensure errors include `requestId`, `traceId`, and top stack frame points to source.

3. Redact PII from error contexts before logging.  
    **Acceptance criteria:**

- A forced error yields a readable stack with original file/line and the request/trace IDs.

* * *

## 19) Load testing & performance budgets

**Goal:** Prevent regressions in latency/throughput.  
**Build steps:**

1. Add a script using `autocannon` to hit `/tasks` with representative query mix.

2. Capture p50/p95/p99 latency and throughput; write a perf budget (e.g., p95 < 120 ms).

3. Add a CI “perf check” job that fails if budget exceeded by N%.  
    **Acceptance criteria:**

- Baseline numbers stored; later regressions fail the perf job.

- Perf report printed as a CI artifact.

* * *

## 20) Developer experience: debug, profiles, and docs

**Goal:** Make the app easy to diagnose locally.  
**Build steps:**

1. Add VS Code `launch.json` to debug with `--inspect`, auto‑attach, and env presets.

2. Add endpoints or CLI triggers to write a CPU profile and heap snapshot (guarded, dev‑only).

3. Document “how to debug/profiling/logs/metrics/traces” in `CONTRIBUTING.md`.  
    **Acceptance criteria:**

- Devs can set breakpoints, generate a CPU profile, and open it in Chrome DevTools.

- The docs explain common workflows end‑to‑end.

* * *

### General expectations (apply to all 20)

- **Security:** Never log secrets or PII; use redaction and structured fields. Sanitize error messages.

- **Stability:** All external calls should have timeouts, retries where appropriate, and be traced/observable.

- **Testing discipline:** Fast unit tests, fewer/slower integration tests, and a pragmatic CI cadence.

- **Documentation:** Every tool (logger, metrics, tracing, CI, hooks) should have a short “how to” in the repo’s README/CONTRIBUTING.
