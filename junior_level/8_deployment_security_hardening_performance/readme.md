# Deployment, Security Hardening, and Performance

Below are **20 “Deployment, Security Hardening, and Performance” build‑style exercises** for a Node.js (JavaScript) backend. They progress from junior to near‑mid level. Prefer **Express**. Keep responses consistent (e.g., `{ data, meta, error }`). Each item includes **step‑by‑step rubrics** and **acceptance criteria**.

* * *

## 1) Minimal, secure Docker image (multi‑stage, non‑root)

**Goal:** Build a small, safe container image.  
**Build steps:**

1. Create a multi‑stage `Dockerfile`: `builder` (installs deps, builds) → `runner` (copies only `dist` + production deps).

2. Use `node:XX-alpine` (or distroless) for `runner`.

3. Create a non‑root user/group, `USER node`, set `WORKDIR /app`, ensure file ownership (`chown -R node:node`).

4. Set `NODE_ENV=production`, `PORT=3000`, `ENVIRONMENT=production`.

5. Add a `HEALTHCHECK` hitting `/livez`.  
    **Acceptance criteria:**

- `docker image inspect` shows single small final image (<300 MB typical with alpine).

- `node` user is active (`id -u` ≠ 0 inside container).

- `docker ps` shows `healthy` after boot.

* * *

## 2) Compose stack (app + Postgres + Redis) with health dependencies

**Goal:** Local prod‑like orchestration.  
**Build steps:**

1. Create `docker-compose.yml` with services: `app`, `db`, `redis`.

2. Add named volumes for DB data; isolated network.

3. Define healthchecks (`pg_isready`, `redis-cli PING`, `curl /readyz`) and `depends_on: condition: service_healthy`.

4. Parameterize via `.env` (ports, passwords).  
    **Acceptance criteria:**

- `docker compose up` starts services; `app` waits until DB/Redis healthy.

- Stopping `db` flips `/readyz` to `503`.

* * *

## 3) Image and dependency supply‑chain guard (SBOM + scan)

**Goal:** Know what you ship and block known vulns.  
**Build steps:**

1. Generate an SBOM (e.g., CycloneDX) during CI for `node_modules` and the image layers.

2. Run a vulnerability scan for **app deps** and **image OS packages**.

3. Fail CI on high severity findings unless explicitly waived with justification.  
    **Acceptance criteria:**

- CI artifact contains SBOM file.

- A seeded high‑severity dependency triggers CI failure until updated or waived.

* * *

## 4) Secrets management & runtime injection

**Goal:** Keep secrets out of images and code.  
**Build steps:**

1. Remove `.env` from production images; read secrets from environment or mounted secrets files.

2. Add a boot‑time validator that fails fast if required vars are missing.

3. Redact secrets from logs; verify no secrets in `/diagz`.  
    **Acceptance criteria:**

- Running without a required secret exits non‑zero with a clear message.

- Grep of logs and `/diagz` shows no secret values.

* * *

## 5) Readiness, liveness, and Docker/K8s health integration

**Goal:** Safe rollouts and restarts.  
**Build steps:**

1. `/livez`: event‑loop responsive; `/readyz`: checks DB/Redis/migrations/queue consumer.

2. Wire `HEALTHCHECK` (Docker) or `readinessProbe/livenessProbe` (K8s) to these endpoints.

3. Make `/readyz` fail (`503`) on dependency outage; do not crash the process.  
    **Acceptance criteria:**

- Probes show `healthy`/`unhealthy` transitions correctly.

- Load balancer removes pod/container from rotation when `/readyz` fails.

* * *

## 6) Graceful shutdown & connection draining

**Goal:** No dropped requests during deploys.  
**Build steps:**

1. Trap `SIGTERM`/`SIGINT`: stop accepting new connections; wait for in‑flight to finish with a timeout (e.g., 30 s).

2. Close DB/Redis/queue clients; flush logs.

3. Exit with code `0` if clean; non‑zero on timeout.  
    **Acceptance criteria:**

- Concurrent requests during shutdown complete successfully (no 499/502).

- Logs show ordered lifecycle: “stop accepting” → “drained” → “closed”.

* * *

## 7) Resource limits & Node process settings

**Goal:** Predictable memory/CPU and timeouts.  
**Build steps:**

1. Set container limits/requests (Compose `deploy.resources` or K8s `resources`) and Node `--max-old-space-size` if needed.

2. Configure server timeouts: `headersTimeout`, `requestTimeout`, `keepAliveTimeout` to sane values.

3. Deny oversized bodies with `express.json({ limit: '1mb' })`.  
    **Acceptance criteria:**

- Load tests respect timeouts; oversize body returns `413`.

- Container OOM is avoided under normal load; memory plateau visible.

* * *

## 8) Reverse proxy with TLS, compression & HSTS

**Goal:** Secure edge, efficient transfer.  
**Build steps:**

1. Add NGINX/Caddy in front of app (container) for TLS termination.

2. Enable gzip/br compression for text responses; set `HSTS` (opt‑in for local).

3. Proxy only whitelisted headers; preserve `X-Forwarded-*`; set `trust proxy` in app.  
    **Acceptance criteria:**

- HTTPS works; `curl -I` shows `Strict-Transport-Security`.

- Compressed responses smaller than raw; app sees correct `req.ip` from proxy.

* * *

## 9) Security headers & strict Content‑Security‑Policy

**Goal:** Reduce XSS/mixed content risks.  
**Build steps:**

1. Enable `helmet` and configure CSP with **nonce** or **sha256** for any inline scripts.

2. Add `Referrer-Policy`, `Permissions-Policy`, `X-Content-Type-Options`, `frame-ancestors`.

3. Provide a minimal doc page to validate CSP behavior.  
    **Acceptance criteria:**

- `/healthz` returns expected headers.

- A test inline script without nonce is blocked (visible in browser devtools).

* * *

## 10) Dependency hygiene (lockfile, audit, Renovate)

**Goal:** Keep deps current and safe.  
**Build steps:**

1. Commit lockfile; enable `npm ci` in CI.

2. Add `npm audit` (or equivalent) gate with severity threshold.

3. Configure Renovate (or similar) to open PRs with grouped updates and automerge for dev‑deps.  
    **Acceptance criteria:**

- CI fails on seeded vulnerable version until updated.

- Renovate opens an update PR that passes tests.

* * *

## 11) Edge abuse controls (rate limit, slowloris, body limits)

**Goal:** Basic DoS resilience at app layer.  
**Build steps:**

1. Add `express-rate-limit` per IP for sensitive routes; add global limiter with burst + sustained limits.

2. Set `server.headersTimeout` < `server.keepAliveTimeout` to mitigate slowloris.

3. Block extremely long URLs and excessive header sizes (reverse proxy + app checks).  
    **Acceptance criteria:**

- 61st request/min on protected endpoint → `429` with `Retry-After`.

- Artificial slow headers connection is dropped per timeout.

* * *

## 12) Structured logs to stdout + shipper compatibility

**Goal:** Deploy‑friendly logs.  
**Build steps:**

1. Emit NDJSON via `pino`, one event per line; include `level`, `time`, `msg`, `requestId`, `traceId`.

2. Avoid rotating logs in the app; rely on platform log collection.

3. Add a field `service.version` from build metadata.  
    **Acceptance criteria:**

- Logs parse cleanly with a standard collector (line‑delimited JSON).

- Each request yields a correlated ID across lines.

* * *

## 13) Performance profiling & flamegraphs (clinic/0x)

**Goal:** Find hot paths and fix them.  
**Build steps:**

1. Run a short load test (e.g., 60 s, `/tasks`) while profiling with `clinic` or `0x`.

2. Identify two hotspots (e.g., synchronous JSON parsing, excessive logging).

3. Optimize (e.g., cache, async, reduce logs), re‑profile, and record deltas.  
    **Acceptance criteria:**

- p95 latency improves by a measurable percentage (define target, e.g., ≥15%).

- Flamegraph shows reduced time in former hotspots.

* * *

## 14) HTTP performance knobs (keep‑alive, caching, compression)

**Goal:** Reduce latency and bandwidth.  
**Build steps:**

1. Ensure keep‑alive enabled; tune `maxSockets`/`maxFreeSockets` for outbound agents.

2. Add `Cache-Control` + ETags for static and cacheable JSON; enable compression for text.

3. Document which routes are cacheable and why.  
    **Acceptance criteria:**

- Repeat GETs for cacheable route return `304` with `If-None-Match`.

- Bandwidth and TTFB drop for compressed endpoints.

* * *

## 15) Database performance (pooling, prepared statements, indexes)

**Goal:** Increase throughput safely.  
**Build steps:**

1. Use a connection pool with sane min/max; verify leak detection.

2. Add prepared statements/parameterized queries for hot endpoints.

3. Add/verify missing indexes supporting common filters/sorts; avoid N+1 queries.  
    **Acceptance criteria:**

- Throughput (RPS) increases under load without saturating the DB.

- DB CPU drops or query latency median improves; no leaked connections after tests.

* * *

## 16) External connection pooling / proxy (e.g., PgBouncer)

**Goal:** Protect DB from connection storms.  
**Build steps:**

1. Add PgBouncer (or similar) container and point the app to it.

2. Pick pooling mode (transaction) and set pool sizes.

3. Measure and compare connection counts and latencies pre/post.  
    **Acceptance criteria:**

- Peak connections on DB reduced significantly; app sees stable latency.

- No functional regressions (transactions still correct).

* * *

## 17) Autoscaling signals & performance budgets

**Goal:** Scale to demand without thrash.  
**Build steps:**

1. Define SLOs and budgets (e.g., p95 < 120 ms, error rate < 1%).

2. Emit metrics for CPU, memory, RPS, queue depth, and p95/p99 latency.

3. Configure HPA (or similar) to scale on CPU **and** custom latency metric (or queue depth).  
    **Acceptance criteria:**

- Under synthetic load, replicas scale up until p95 meets budget.

- Scale‑down occurs only after cooldown; no oscillation.

* * *

## 18) Blue/green + canary with feature flags

**Goal:** Risk‑reduced deploys.  
**Build steps:**

1. Prepare blue/green environments behind a router; promote green when `/readyz` passes.

2. Add a canary route or weighted routing to send 5–10% traffic to new version.

3. Gate new code paths with feature flags; support instant rollback.  
    **Acceptance criteria:**

- Canary receives the configured fraction and metrics isolate it.

- Rollback returns all traffic to stable version within minutes.

* * *

## 19) Backup, restore, and disaster recovery drill

**Goal:** Prove you can recover data.  
**Build steps:**

1. Automate DB backups (daily full + WAL/incrementals); encrypt at rest; store off‑site.

2. Document RPO/RTO targets.

3. Run a drill: restore to a fresh environment and repoint the app.  
    **Acceptance criteria:**

- Recovery completes within the documented RTO; data loss within RPO.

- App functions correctly against the restored DB; checksums/row counts match expectations.

* * *

## 20) Chaos experiment: dependency outages & network faults

**Goal:** Verify resilience under failure.  
**Build steps:**

1. Inject failures: kill Redis, add latency/packet loss to provider calls, drop DB for 30 s.

2. Ensure circuit breakers, timeouts, and fallbacks (stale‑if‑error) engage.

3. Emit alerts once per window; auto‑recover when dependencies return.  
    **Acceptance criteria:**

- During outage, app returns degraded or cached responses where documented; avoids total crash.

- After recovery, breakers close and metrics normalize without manual intervention.

* * *

### General expectations (apply to all 20)

- **Security:** Non‑root containers, minimal images, no secrets in images or logs, strict headers, validated inputs, rate limits.

- **Reliability:** Timeouts + retries with backoff, circuit breakers, graceful shutdown, readiness gates.

- **Performance:** Set budgets (p95/p99), profile before/after, use pooling/caching/compression/ETags, and avoid N+1.

- **Operations:** Everything observable (logs/metrics/traces), CI gates for SBOM/vulns, and documented runbooks (deploy, rollback, DR).

- Add minimal **README**: how to run, env vars, and curl examples for each exercise.

- If you want, turn the acceptance criteria into **automated tests with autocannon test scripts**,
