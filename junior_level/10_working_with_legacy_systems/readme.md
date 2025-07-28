# Working with Legacy Systems

Below are **20 “Working with Legacy Systems” build‑style exercises** for Node.js (JavaScript) that progress from easy to near mid‑level. Prefer **Express**. Suggested libs: HTTP (**got/axios**), SOAP/XML (**soap**, **fast-xml-parser**), feature flags (**unleash**, **launchdarkly** or a simple in‑house flag store), schema tools (**knex**, **sequelize**, **pg**), testing (**Jest**, **supertest**, **nock**), contract testing (**Pact**, **Dredd/Prism**), queues (**BullMQ + Redis**), caching (**ioredis**), and tracing/logging (**pino**, **OpenTelemetry**).  
Use a consistent response envelope (e.g., `{ data, meta, error }`), stable error codes, and never echo secrets.

* * *

## 1) “Adapter first” bootstrap around a legacy service

**Goal:** Stand up a thin Node façade that forwards requests to a legacy HTTP service unchanged.  
**Build steps:**

1. Create `GET /legacy/*` that proxies to `LEGACY_BASE_URL` with path/query passthrough.

2. Forward headers selectively (whitelist), inject `X-Request-Id`.

3. Return upstream status/body verbatim, but always wrap in your envelope on success/errors.  
    **Acceptance criteria:**

- `/legacy/ping` returns exactly the legacy payload under `{ data }`.

- Missing legacy host → `502` with `{ error.code:'LEGACY_UNAVAILABLE' }`.

- Logs show `requestId` and upstream latency.

* * *

## 2) Characterization tests (Golden Master) for a legacy endpoint

**Goal:** Lock current behavior before refactoring.  
**Build steps:**

1. Record N sample responses from the legacy `/orders/:id` for a fixed corpus (use `nock` fixtures or a capture tool).

2. Write tests that call your adapter and snapshot **semantically** (stable fields only).

3. Mask volatile fields (timestamps, IDs) before snapshotting.  
    **Acceptance criteria:**

- Changing adapter logic that alters responses breaks tests.

- Snapshots ignore whitelisted fields (no false positives).

* * *

## 3) Anti‑corruption layer (ACL) DTO mapping

**Goal:** Translate legacy shapes to clean domain DTOs.  
**Build steps:**

1. Create mappers: `legacyOrder → OrderDTO` (rename fields, normalize types, fix casing).

2. Apply at the façade: `/orders/:id` returns `OrderDTO` and hides legacy quirks.

3. Validate with Zod/Ajv to guarantee DTO shape.  
    **Acceptance criteria:**

- Fields with inconsistent types in legacy (string numbers) become typed numbers in DTO.

- Unknown/extra legacy fields are ignored; missing required → `502` with `code:'LEGACY_CONTRACT_VIOLATION'`.

* * *

## 4) Strangler‑Fig routing (one route at a time)

**Goal:** Replace a legacy endpoint with a new implementation behind the same URL.  
**Build steps:**

1. Add router that decides: if `X‑Use‑New: 1` (or flag on), route to `newOrdersController`; else proxy legacy.

2. Keep outward contract identical (DTO from #3).

3. Log split ratios and correctness checks.  
    **Acceptance criteria:**

- With flag off: legacy path used (verified by logs).

- With flag on: new path used and payloads are contract‑equivalent for the same request.

* * *

## 5) Feature flags & kill switch

**Goal:** Toggle new code safely and roll back fast.  
**Build steps:**

1. Add a flag provider (file/env/Unleash). Cache flags with short TTL.

2. Implement a **kill switch** flag that forces all traffic to legacy.

3. Expose `/ops/flags` (secured) to read current flag states.  
    **Acceptance criteria:**

- Toggling `orders_v2_enabled` changes routing within 5 seconds.

- Kill switch overrides all other flags.

* * *

## 6) Tolerant reader: envelope stabilization

**Goal:** Prevent breakage when legacy adds fields.  
**Build steps:**

1. Define a strict minimal contract you depend on; ignore unknown properties.

2. Add a translator that defaults missing optional fields and normalizes enums.

3. Return a `meta.unknownFields` counter for observability.  
    **Acceptance criteria:**

- New fields from legacy don’t break your parser.

- Missing optional field results in a defaulted value; missing required → mapped `502`.

* * *

## 7) Legacy DB integration via repository with quirks

**Goal:** Read from a legacy database schema without leaking it.  
**Build steps:**

1. Create `ordersRepo` with SQL tailored to legacy constraints (snake\_case, denormalized columns).

2. Map row → `OrderDTO` in the repo, not in controllers.

3. Add read‑only role/connection; parameterize all queries.  
    **Acceptance criteria:**

- Controllers never contain SQL.

- Wrong parameter type returns `400`; SQL injection attempts fail safely.

* * *

## 8) Contract tests against legacy API

**Goal:** Detect upstream changes early.  
**Build steps:**

1. Write a Pact/Dredd suite defining the **legacy** contract you expect.

2. Run in CI daily against a staging legacy server.

3. Fail on breaking changes; notify team with actionable diff.  
    **Acceptance criteria:**

- Adding/removing fields in legacy that change the contract fails the job.

- The failure output highlights which paths/fields broke.

* * *

## 9) SOAP/XML ↔ REST/JSON translator

**Goal:** Integrate with a legacy SOAP service through a clean REST API.  
**Build steps:**

1. Expose `POST /payments/charge` (JSON), transform to SOAP envelope and call legacy.

2. Parse XML response, map to DTO, and return JSON.

3. Handle SOAP faults and map to 4xx/5xx with stable error codes.  
    **Acceptance criteria:**

- Valid request hits SOAP service and returns normalized JSON.

- SOAP fault `InsufficientFunds` → `402 PAYMENT_REQUIRED` with `{ error.code:'INSUFFICIENT_FUNDS' }`.

* * *

## 10) CSV batch import with messy formats

**Goal:** Consume legacy exports with headers that change per environment.  
**Build steps:**

1. Accept `text/csv`; detect delimiter, BOM, and encoding; normalize to UTF‑8.

2. Build a header mapping layer (aliases) and a row validator with detailed error lines.

3. Process in chunks; report per‑row success/failure.  
    **Acceptance criteria:**

- Mis‑encoded input is normalized; invalid rows are skipped with precise line numbers.

- Import report totals match processed rows; partial failure returns `207 Multi‑Status` (or `200` with details).

* * *

## 11) Idempotency for duplicate messages/requests

**Goal:** Make legacy‑integration endpoints safe for retries.  
**Build steps:**

1. Support `Idempotency-Key` on `POST /orders`.

2. Store seen keys with result hash (Redis) for 24h; return the same response for repeats.

3. Ensure downstream (legacy) is not called twice for the same key.  
    **Acceptance criteria:**

- Replaying the same key avoids duplicate legacy calls and returns identical payload/status.

- New key triggers a real call.

* * *

## 12) Transactional outbox for eventual consistency

**Goal:** Publish events reliably when legacy DB is the source of truth.  
**Build steps:**

1. On write to legacy DB (or your mirror), record a row in `outbox` within the same transaction.

2. A worker reads `outbox`, publishes to Redis/Kafka, and marks rows sent.

3. Ensure **exactly‑once** semantics via idempotent consumer and unique message IDs.  
    **Acceptance criteria:**

- Killing the worker mid‑publish does not lose events; they are retried.

- Downstream gets each event exactly once (no duplicates observed by consumer).

* * *

## 13) Progressive schema migration using views

**Goal:** Change the contract without immediate table changes.  
**Build steps:**

1. Create DB views that expose new column names/types mapped from legacy.

2. Point your repo to views; backfill new columns gradually.

3. Add a migration plan and a rollback recipe.  
    **Acceptance criteria:**

- Reads come from views with the new contract; legacy tables remain intact.

- Toggling back to tables via config works without code changes.

* * *

## 14) Retry/circuit‑breaker policies specific to legacy

**Goal:** Encapsulate flaky behavior of legacy systems.  
**Build steps:**

1. Wrap all legacy calls in a **circuit breaker** (opossum) with tuned thresholds per endpoint.

2. Add bounded retries with exponential backoff + jitter for retryable errors only.

3. Expose `/integrations/legacy/breaker` with state metrics.  
    **Acceptance criteria:**

- Repeated timeouts open the breaker; subsequent calls short‑circuit.

- Non‑retryable errors are not retried; mapped to stable error codes.

* * *

## 15) Observability retrofits: logs, metrics, traces

**Goal:** Gain visibility into legacy interactions.  
**Build steps:**

1. Add `pino` structured logs with `requestId`, `legacyCorrelationId`, and timings.

2. Add Prometheus metrics: latency histograms and error counters by legacy endpoint.

3. Add OpenTelemetry spans wrapping each legacy call.  
    **Acceptance criteria:**

- `/metrics` shows per‑endpoint histograms and error counts.

- Traces contain child spans for each legacy call with status tags.

* * *

## 16) Caching front for slow legacy endpoints

**Goal:** Reduce latency and load.  
**Build steps:**

1. Implement read‑through Redis cache on expensive `GET /catalog` with per‑tenant keys.

2. Add **stale‑while‑revalidate** for 2 minutes; invalidate on related writes.

3. Respect `Vary` semantics (e.g., `Accept-Language`) in keys.  
    **Acceptance criteria:**

- First call MISS, second HIT within TTL; during legacy outage, stale content is served with `meta.cache.state='stale'`.

- Writes invalidate relevant keys.

* * *

## 17) Security hardening around legacy inputs/outputs

**Goal:** Sanitize before passing to/receiving from legacy.  
**Build steps:**

1. Validate and sanitize all inputs (lengths, encodings, Unicode normalization, CR/LF stripping).

2. Enforce strict `Content-Security-Policy` and set `X-Content-Type-Options` for any file responses proxied.

3. Redact secrets; block path traversal and SSRF to arbitrary legacy hosts.  
    **Acceptance criteria:**

- Malformed input is rejected with `400` and localized error messages.

- Attempted SSRF (changing base URL) is denied with `403`.

* * *

## 18) Shadow traffic (parallel run) & diffing

**Goal:** Safely evaluate the new implementation.  
**Build steps:**

1. For selected endpoints, **duplicate** requests to new code asynchronously (do not affect user response).

2. Diff legacy vs new responses (field‑by‑field, order‑insensitive); log discrepancies.

3. Expose `/ops/shadow-stats` with match rates.  
    **Acceptance criteria:**

- Live traffic continues to use legacy responses; shadow diffs accumulate.

- Mismatch rate visible; a threshold alert triggers when exceeding X%.

* * *

## 19) Data reconciliation tool & drift reports

**Goal:** Detect divergence between systems.  
**Build steps:**

1. Build a CLI/endpoint to sample N records from legacy and from your store, compare hashes of canonical DTOs.

2. Produce a report grouped by mismatch reason (missing, stale, value diff).

3. Store history for trend analysis.  
    **Acceptance criteria:**

- Running reconciliation yields accurate counts and exemplars.

- Report export (CSV/JSON) is downloadable and includes timestamps.

* * *

## 20) Decommission checklist & cutover runbook

**Goal:** Finish the strangler and retire legacy safely.  
**Build steps:**

1. Create a checklist: traffic migration %, error rates within SLO, parity tests green, data backfills done, DR docs updated.

2. Implement final switch flag `legacy_disabled=true` and 404/410 routes for any undisplaced endpoints.

3. Write rollback steps and time‑boxed guardrails.  
    **Acceptance criteria:**

- With all gates green, turning `legacy_disabled` serves only new implementation; legacy traffic drops to 0.

- Rolling back instructions restore previous routing within minutes.

* * *

### General expectations (apply to all 20)

- **Safety first:** Characterize and test before changing behavior; prefer **add‑then‑switch** over **in‑place edits**.

- **Bounded change:** Introduce **seams** (adapters, views, feature flags) and change one seam at a time.

- **Contracts:** Stabilize via DTOs/ACL; validate aggressively at boundaries; maintain **stable error codes**.

- **Resilience:** Timeouts, retries with backoff, and circuit breakers around every legacy call; never block the event loop.

- **Observability:** Correlate logs by `requestId` and legacy IDs; export histograms, error rates, and breaker states; document runbooks.

- **Security:** Treat legacy I/O as untrusted; sanitize inputs/outputs, restrict outbound hosts, and redact secrets.

- **Documentation:** Keep a risk register, migration plan, and cutover/rollback runbooks in the repo; update OpenAPI and ADRs as contracts evolve.
