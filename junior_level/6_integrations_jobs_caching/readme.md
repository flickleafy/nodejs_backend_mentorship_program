# Integrations, Jobs, and Caching

Below are **20 “Integrations, Jobs, and Caching” build‑style exercises** for a Node.js (JavaScript) backend. They progress from junior to near‑mid level. Prefer **Express**. Suggested libs: HTTP (**got/axios**), mocking (**nock**), circuit breaker (**opossum**), queue (**BullMQ + Redis**), scheduler (**node-cron**), cache/lock (**ioredis**, **redlock**), retry (**p-retry**), metrics (**prom-client**), tracing (**@opentelemetry/api**).  
Use a consistent response envelope (e.g., `{ data, meta, error }`) and never log secrets.

* * *

## 1) Third‑party API call with timeout & retries

**Goal:** Call an external weather API reliably.  
**Build steps:**

1. Create `GET /weather?city=` that calls an external endpoint (mock with `nock`).

2. Use `got` with **connection/request timeout** (e.g., 2 s) and **exponential backoff** retries for `ETIMEDOUT`, `5xx`.

3. Map upstream errors to your own error format; include `providerCorrelationId` when available.  
    **Acceptance criteria:**

- When upstream delays > 2 s → your route returns `504` after max retries.

- Transient 500 once, then 200 → your route returns `200` with parsed JSON.

- Tests assert retry count and backoff (via `nock` call counters).

* * *

## 2) Circuit breaker around the external call

**Goal:** Fail fast when the provider is unhealthy.  
**Build steps:**

1. Wrap the call from #1 using **opossum** with sensible thresholds (error rate, rolling window, half‑open probe).

2. Expose `GET /integrations/weather/breaker` to show breaker state and stats.

3. Emit logs/metrics on `open`, `halfOpen`, `close`.  
    **Acceptance criteria:**

- After repeated failures, breaker enters **OPEN**; subsequent requests short‑circuit (no upstream hit).

- Half‑open allows a probe; success closes breaker.

- `/breaker` shows accurate state transitions.

* * *

## 3) OAuth2 client‑credentials token management

**Goal:** Call a provider that requires OAuth2 and cache tokens.  
**Build steps:**

1. Implement client‑credentials grant to obtain `access_token` with expiry.

2. Cache token in Redis with TTL; refresh **5% before** expiry; avoid refresh stampede via a Redis lock.

3. Inject `Authorization: Bearer` header on downstream calls.  
    **Acceptance criteria:**

- Only one token request occurs across concurrent callers.

- Token auto‑refreshes before expiry; no 401s from provider in happy path.

- If refresh fails, subsequent calls return 502 with stable error code.

* * *

## 4) External API pagination & provider rate limits

**Goal:** Respect `429` and `Retry‑After`.  
**Build steps:**

1. Build `GET /partners/items` that aggregates all pages from provider (server‑side).

2. On `429` or `Retry‑After`, **pause** and retry accordingly.

3. Surface partial results only when `?partial=true` is passed, else fail atomically.  
    **Acceptance criteria:**

- Provider returns `429` with `Retry‑After: 2` → your code waits and completes successfully.

- Without `?partial=true`, a failure on page 3/5 returns `502`; with it, returns pages 1–2 and a warning in `meta`.

* * *

## 5) Webhook ingestion with HMAC verification

**Goal:** Secure inbound webhooks.  
**Build steps:**

1. `POST /webhooks/provider-x` accepts JSON bodies; compute `HMAC‑SHA256` over the raw body with shared secret; compare to `X‑Signature` and `X‑Timestamp`.

2. Enforce **timestamp skew** window (±5 min) and **constant‑time** comparison.

3. On success, enqueue a job (see BullMQ exercises).  
    **Acceptance criteria:**

- Invalid signature or stale timestamp → `401` (do not process).

- Valid request enqueues exactly one job and returns `202`.

- Replay of the same payload+timestamp is rejected (`409` with `code:'REPLAY_DETECTED'`), see #6.

* * *

## 6) Idempotent webhooks & replay protection

**Goal:** Process each delivery once.  
**Build steps:**

1. Read `X‑Delivery‑Id`; store seen IDs in Redis with TTL (e.g., 7 days).

2. If seen, return `200` (or `409`, choose and document) without re‑enqueuing.

3. Include the dedup decision in logs/metrics.  
    **Acceptance criteria:**

- First call enqueues job; second with same `X‑Delivery‑Id` does **not**.

- TTL expiration allows reprocessing only after window (documented).

* * *

## 7) Outbound webhooks with retries & DLQ

**Goal:** Notify partners reliably.  
**Build steps:**

1. Create `webhooks:outbound` queue; a worker POSTs to partner endpoints.

2. Use exponential backoff, cap attempts, and move failed jobs to a **DLQ** queue.

3. Provide `/admin/webhooks/dlq` to list/replay items.  
    **Acceptance criteria:**

- Transient partner failures are retried, then succeed.

- Permanent failures end in DLQ with error metadata.

- “Replay” moves an item back to the main queue and succeeds.

* * *

## 8) Job queue for emails (BullMQ + Redis)

**Goal:** Offload email sending.  
**Build steps:**

1. Create `emails:welcome` queue and worker; mock the provider (no real send).

2. Expose `POST /emails/welcome` to enqueue `{ userId, email }`.

3. Configure concurrency, backoff, and **job idempotency** (`jobId = userId`).  
    **Acceptance criteria:**

- Enqueuing the same user twice does not create duplicate jobs.

- Worker logs success; failures retry per policy; DLQ on repeated failures.

* * *

## 9) Scheduled jobs with node‑cron (singleton)

**Goal:** Nightly archival without duplicates.  
**Build steps:**

1. Schedule a nightly job to archive `done=true` tasks older than 30 days.

2. Ensure **single instance** across multiple app replicas via Redis lock (e.g., Redlock) before running.

3. Emit a summary report (counts) to logs/metrics.  
    **Acceptance criteria:**

- In multi‑process test, only one instance runs per tick.

- Job respects timezone and schedule; metrics show counts archived.

* * *

## 10) Batch processing with progress tracking

**Goal:** Track long‑running jobs.  
**Build steps:**

1. `POST /reports/run` enqueues a multi‑step job (e.g., export tasks to CSV).

2. Store job progress (0–100) and status (`queued|running|done|failed`) in Redis.

3. `GET /reports/:jobId` returns status, progress, and download link when ready.  
    **Acceptance criteria:**

- Polling shows progress increasing; final state exposes a valid download URL.

- Failure exposes `error.message` and no download link.

* * *

## 11) Read‑through cache for list endpoints

**Goal:** Speed up hot reads; invalidate on writes.  
**Build steps:**

1. Cache `GET /tasks` by **user + query hash** for 30 s in Redis.

2. On `POST/PATCH/DELETE /tasks*`, **invalidate** keys for that user.

3. Use compressed values (e.g., `gzip`) and set `maxmemory-policy allkeys-lru`.  
    **Acceptance criteria:**

- First read is a MISS; next read within TTL is a HIT (verify via metrics/logs).

- Writes invalidate and next read rebuilds cache (MISS→HIT).

* * *

## 12) Request coalescing (single‑flight)(de‑duplication)

**Goal:** Avoid thundering herd on cache misses.  
**Build steps:**

1. Implement a per‑key **in‑process** and **distributed** lock (Redis) to ensure only one request rebuilds cache.

2. Other concurrent callers await the same promise; set a short “rebuild” timeout.  
    **Acceptance criteria:**

- 20 concurrent identical GETs on a cold key trigger **one** upstream query.

- All callers receive the same response; none time out in the happy path.

* * *

## 13) Stale‑while‑revalidate & stale‑if‑error

**Goal:** Favor availability under load/outages.  
**Build steps:**

1. Serve cached data if fresh; if stale (e.g., <= 2 min), serve **stale** immediately and refresh in background.

2. If upstream errors, serve **stale‑if‑error** up to 10 min and add a `meta.cache: { state:'stale', age }`.  
    **Acceptance criteria:**

- Stale cache served instantly, followed by background refresh (verify updated cache on the next call).

- During upstream outage, clients still get stale data with clear metadata.

* * *

## 14) Distributed locks with Redlock

**Goal:** Guarantee once‑only operations across replicas.  
**Build steps:**

1. Use **Redlock** to protect critical sections (e.g., nightly aggregation, token refresh).

2. Set sensible lock TTL and jittered retries to avoid herd effects.

3. Handle lock acquisition failure gracefully (skip with warning).  
    **Acceptance criteria:**

- With two workers racing, only one acquires the lock.

- If a worker crashes mid‑section, lock eventually expires and next run succeeds.

* * *

## 15) Provider rate budgeting (outbound limiter)

**Goal:** Keep within partner quotas.  
**Build steps:**

1. Implement a **distributed sliding‑window** or **token bucket** in Redis for the provider (e.g., 60 req/min).

2. If budget exceeded, either queue the call or return 429 with `Retry‑After`.  
    **Acceptance criteria:**

- 100 calls burst → only 60 pass within the first minute; remainder delayed or rejected per policy.

- Headers reflect remaining quota and reset time.

* * *

## 16) Batch/aggregate upstream requests

**Goal:** Reduce chattiness.  
**Build steps:**

1. Accept `GET /profiles?ids=a,b,c`. Buffer IDs for 10–20 ms and issue **one** batch request to provider (fan‑in).

2. Cache results by id; return in the original order; mark misses.  
    **Acceptance criteria:**

- 10 concurrent single‑ID calls within the window cause exactly **one** upstream batch.

- Response preserves client order and includes `meta.hits/misses`.

* * *

## 17) Backpressure & overload protection

**Goal:** Stay stable under load.  
**Build steps:**

1. Track queue depth, event loop lag, and in‑flight requests.

2. When thresholds exceed limits, return `503` with `Retry‑After` or shed non‑critical work (feature flag).

3. Expose `/system/capacity` for current headroom.  
    **Acceptance criteria:**

- Synthetic overload triggers `503` with proper headers.

- When load drops, service resumes normal responses.

* * *

## 18) Observability for integrations & jobs

**Goal:** Make issues visible.  
**Build steps:**

1. Add **prom‑client** histograms for downstream latency, success/error counters, cache hits/misses, queue depths.

2. Add **OpenTelemetry** spans for external calls and job executions; propagate `traceId` in outbound requests.

3. `/metrics` endpoint exposes all.  
    **Acceptance criteria:**

- Metrics show non‑zero counts for calls, retries, cache hits/misses.

- Traces include child spans for downstream HTTP and BullMQ jobs with `traceId` correlation.

* * *

## 19) Degradation mode with cached fallback

**Goal:** Keep UX usable during outages.  
**Build steps:**

1. If the breaker (#2) is OPEN, immediately serve **last good** cached response (if available) and set `meta.degraded=true`.

2. Record an alert metric/event the first time degradation activates per window.  
    **Acceptance criteria:**

- With breaker open, responses come from cache with `degraded` flag; no upstream calls occur.

- When provider recovers, system returns to live data and clears the flag.

* * *

## 20) Secret rotation & multi‑key support

**Goal:** Rotate provider API keys without downtime.  
**Build steps:**

1. Support **key versioning** (e.g., `k1`,`k2`) and attempt primary, then secondary on 401s (for a limited window).

2. Store key ids in config; allow hot‑reload or SIGHUP to switch primaries.

3. Log/metric for rotation events; never print secrets.  
    **Acceptance criteria:**

- During rotation, requests continue to succeed without restarts.

- If primary becomes invalid, fallback key is used transparently and you get a rotation alert.

* * *

### General expectations (apply to all 20)

- **Testing:** Use `nock` for outbound HTTP, fake timers for cron, and Redis test containers or an in‑memory alternative when appropriate.

- **Resilience:** Retries are **bounded** with backoff + jitter. Use circuit breakers where appropriate. Always time out external calls.

- **Security:** Validate and sign webhooks; sanitize all inputs; don’t log PII or secrets; restrict outbound domains by allow‑list if possible.

- **Cache discipline:** Define clear **key namespaces**, TTLs, and invalidation rules. Document the filter→sort→paginate order before caching.

- **Operations:** Emit structured logs with `requestId`/`traceId` and **actionable** metrics (SLIs). Provide runbooks for DLQ and breaker resets.

- Add minimal **README**: how to run, env vars, and curl examples for each exercise.

- If you want, turn the acceptance criteria into **automated tests with Supertest/Jest**,
