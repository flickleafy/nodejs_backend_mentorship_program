# Foundations & HTTP

Below are **20 “Foundations & HTTP” build‑style exercises** for Node.js (JavaScript) with **step‑by‑step rubrics** and **acceptance criteria**. They progress from very basic toward the upper end of junior level. Use the native `http` module where noted; otherwise prefer **Express**. You may use small, standard libs (e.g., `dotenv`, `cors`, `express-rate-limit`, `joi`/`express-validator`) when suggested.

* * *

## 1) Native HTTP server + simple healthcheck

**Goal:** Learn the `http` core module and JSON responses.  
**Build steps:**

1. Initialize a Node project (`npm init -y`) and create `server.js`.

2. Use `http.createServer` to listen on port `3000`.

3. If the path is `/health` and method is `GET`, return `200` with `{"status":"ok"}` and header `Content-Type: application/json`.

4. For other paths, return `404` with a small JSON error.  
    **Acceptance criteria:**

- `curl -i http://localhost:3000/health` → `200` and `{"status":"ok"}`.

- `curl -i http://localhost:3000/unknown` → `404` JSON error.

* * *

## 2) Express bootstrap + `/healthz` with uptime

**Goal:** Switch to Express and send structured JSON.  
**Build steps:**

1. Install `express` and create `app.js`.

2. Implement `GET /healthz` returning `{ uptime, timestamp, version }`.

3. `uptime` from `process.uptime()`, `timestamp` as ISO string, `version` from `package.json`.

4. Start on port `3000` via `npm start`.  
    **Acceptance criteria:**

- `curl -s http://localhost:3000/healthz` shows all three fields with correct types/values.

* * *

## 3) Environment configuration with `dotenv`

**Goal:** Centralize config and fail fast when required envs are missing.  
**Build steps:**

1. Install `dotenv` and load it at app start.

2. Require `PORT` and `APP_NAME`; if missing, log a clear error and `process.exit(1)`.

3. Use `PORT` to listen and log `${APP_NAME} listening on PORT`.  
    **Acceptance criteria:**

- Missing envs → process exits with non‑zero and a clear message.

- With `.env` present, server starts and listens on the specified port.

* * *

## 4) Request logging middleware (hand‑rolled)

**Goal:** Understand middleware and response lifecycle timing.  
**Build steps:**

1. Create middleware capturing `method`, `originalUrl`, and `start = process.hrtime.bigint()`.

2. On `res.on('finish')`, compute elapsed ms and `console.log` one line per request.

3. Apply globally before routes.  
    **Acceptance criteria:**

- Every request logs: `METHOD PATH STATUS DURATIONms`.

- Duration increases for artificially delayed routes.

* * *

## 5) Centralized error handler

**Goal:** Standardize error responses.  
**Build steps:**

1. Add route that throws synchronously and another that rejects a Promise.

2. Implement error middleware `(err, req, res, next)` returning JSON `{ error: { message, code } }`.

3. Map known errors to proper HTTP status (e.g., 400, 404); default to 500.  
    **Acceptance criteria:**

- Throwing route returns `500` with JSON body (no stack traces in body).

- Known app error returns mapped status and code.

* * *

## 6) Params, query, and basic i18n greeting

**Goal:** Use route params and query parsing.  
**Build steps:**

1. Add `GET /greet/:name` reading `req.params.name`.

2. Optional `?lang=en|es|pt` changes greeting string.

3. Return `{ message: "Olá, Maria!" }` style output.  
    **Acceptance criteria:**

- `/greet/Ana?lang=pt` returns Portuguese greeting.

- Unknown `lang` falls back to English.

* * *

## 7) JSON body validation on `/echo` (Joi or express-validator)

**Goal:** Validate input and return helpful errors.  
**Build steps:**

1. Enable `express.json()` for JSON parsing.

2. POST `/echo` expects `{ message: string, 1–200 chars }`.

3. On pass: return `{ data: { message } }`; on fail: `400` with field errors.  
    **Acceptance criteria:**

- Missing or too‑long `message` → `400` with explicit error.

- Valid body → `200` and echoes back.

* * *

## 8) Strict CORS policy + explain preflight

**Goal:** Control cross‑origin access and handle OPTIONS.  
**Build steps:**

1. Install `cors` and allow only origin `http://localhost:3000`, methods `GET,POST`.

2. Ensure `OPTIONS` preflight returns allowed methods/headers for `/echo`.

3. Document briefly in README what a preflight is.  
    **Acceptance criteria:**

- Browser requests from allowed origin succeed; others blocked.

- `curl -i -X OPTIONS http://localhost:PORT/echo` shows appropriate CORS headers.

* * *

## 9) Rate limiting `/echo`

**Goal:** Prevent abuse of an endpoint.  
**Build steps:**

1. Install `express-rate-limit` and set `windowMs=1 minute`, `max=60`.

2. Apply limiter only to `/echo`.

3. When limited, return `429` and include `Retry-After`.  
    **Acceptance criteria:**

- 61st request within a minute returns `429` with JSON error and `Retry-After` header.

* * *

## 10) ETag & `Cache-Control` for a static route

**Goal:** Enable client caching.  
**Build steps:**

1. Create `/about` route that returns a static JSON or small HTML.

2. Set `Cache-Control: public, max-age=300`.

3. Ensure ETag is enabled (Express default) and handled correctly on `If-None-Match`.  
    **Acceptance criteria:**

- First `GET /about` → `200` with ETag.

- Subsequent `GET` with `If-None-Match` → `304` (no body).

* * *

## 11) Malformed JSON handling → `400`

**Goal:** Robust body parsing failures.  
**Build steps:**

1. Keep `express.json()` but add an error handler that detects `SyntaxError` from JSON parse.

2. Return `400` with `{ error: { message: "Invalid JSON" } }`.  
    **Acceptance criteria:**

- `curl -i -X POST /echo -d '{"message":123' -H 'Content-Type: application/json'` → `400`.

- Well‑formed JSON still passes validation from exercise #7.

* * *

## 12) Custom `404` (not found) and `405` (method not allowed)

**Goal:** Clear responses for unknown routes/methods.  
**Build steps:**

1. Add a “catch‑all” 404 middleware after all routers.

2. For known routes, reject unsupported methods with `405` and `Allow` header.

3. Return consistent JSON error shapes.  
    **Acceptance criteria:**

- `/unknown` → `404` with `{error:{message:"Not Found"}}`.

- `PUT /healthz` → `405` with `Allow: GET`.

* * *

## 13) Content negotiation with `Accept`

**Goal:** Respect `Accept` header and return `406` when unsupported.  
**Build steps:**

1. For `/healthz`, support `application/json` (default) and `text/plain`.

2. If client requests unsupported type (e.g., `text/xml`), return `406`.

3. Use `req.accepts()` to branch.  
    **Acceptance criteria:**

- `curl -H 'Accept: text/plain' /healthz` returns `text/plain`.

- `Accept: text/xml` → `406`.

* * *

## 14) Normalize trailing slashes & case sensitivity

**Goal:** Avoid duplicate route variants.  
**Build steps:**

1. Configure Express with `caseSensitiveRouting: false`, `strictRouting: false` **or** add a middleware redirecting `/path/` → `/path`.

2. Ensure canonical URLs (no trailing slash).  
    **Acceptance criteria:**

- `/healthz/` redirects (301/308) or serves same resource consistently.

- `/HeAlThZ` still resolves (if case‑insensitive configured).

* * *

## 15) Request ID correlation header

**Goal:** Trace a single request across logs.  
**Build steps:**

1. Middleware to read `X-Request-Id` or generate a UUID if missing.

2. Attach the ID to `res.locals.requestId`, response header, and logs from exercise #4.

3. Include the ID in error responses.  
    **Acceptance criteria:**

- Response includes `X-Request-Id`.

- Logs show the same ID; error responses include `requestId` field.

* * *

## 16) Environment profiles via `NODE_ENV`

**Goal:** Behavior differs between development and production.  
**Build steps:**

1. Read `NODE_ENV` and set `isDev = NODE_ENV !== 'production'`.

2. Enable verbose request logs only in dev; concise in prod.

3. In prod, disable detailed error messages (no stack traces in body).  
    **Acceptance criteria:**

- With `NODE_ENV=development`, logs are verbose; with `production`, they’re concise.

- Error payloads in prod do not expose stack traces.

* * *

## 17) Static file serving with safe defaults

**Goal:** Serve assets correctly and cache them.  
**Build steps:**

1. Add `/public` folder with `robots.txt` and a small `index.html`.

2. Serve with `express.static('public', { etag:true, maxAge:'1h', immutable:true })`.

3. Block dotfiles and disable directory listings (default in Express).  
    **Acceptance criteria:**

- `GET /robots.txt` works with cache headers.

- Hidden files (e.g., `.env`) are not served.

* * *

## 18) Liveness vs readiness endpoints

**Goal:** Differentiate “process is up” vs “app is ready”.  
**Build steps:**

1. `GET /livez` returns `200` if the process handles requests (no checks).

2. `GET /readyz` returns `200` only if required config exists (reuse #3), and a lightweight dependency check passes (e.g., can read `package.json` or resolve DNS for `localhost`).

3. Return `503` when checks fail.  
    **Acceptance criteria:**

- Break the readiness condition (e.g., remove an env var) → `/readyz` returns `503` but `/livez` still `200`.

* * *

## 19) Router modularization

**Goal:** Organize endpoints with `express.Router`.  
**Build steps:**

1. Create `/api` router and move `/echo`, `/greet/:name` into `/api`.

2. Keep `/healthz`, `/livez`, `/readyz` at root.

3. Ensure error and 404 handlers still work after refactor.  
    **Acceptance criteria:**

- Old routes moved under `/api` respond identically aside from the new base path.

- No regression in logging, CORS, validation, or errors.

* * *

## 20) Dev workflow with `nodemon` + clean startup/shutdown logs

**Goal:** Faster iteration and predictable lifecycle messages.  
**Build steps:**

1. Install `nodemon` and add `npm run dev` with a `nodemon.json` that restarts on `*.js`.

2. On startup, log app name, version, env, and bound URL.

3. Handle `SIGINT` (Ctrl‑C) to log “Shutting down…” then `server.close()` and exit cleanly.  
    **Acceptance criteria:**

- Changing a file triggers reload in dev.

- `Ctrl‑C` prints shutdown message, closes server, and exits with code 0.

* * *

### Notes for students

- Keep responses **consistent**: Prefer a small envelope like `{ data, error, meta }` for success/failure (ex. #5).

- Prefer **explicit status codes**, correct `Content-Type`, and small, clear error messages.

- Organize code by **Separation of Concerns**: `app.js` (composition), `routes/`, `middleware/`, `config/`, `lib/`.

- Add minimal **README**: how to run, env vars, and curl examples for each exercise.

- If you want, turn the acceptance criteria into **automated tests with Supertest/Jest**,
