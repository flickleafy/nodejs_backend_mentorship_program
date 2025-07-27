# Files, Validation, and Documentation

Below are **20 “Files, Validation, and Documentation” build‑style exercises** for Node.js (JavaScript) that progress from easy to near mid‑level. Prefer **Express**. Suggested libs in parentheses: file upload (**multer/busboy**), image processing (**sharp**), type detection (**file-type**), validation (**zod/ajv/express‑validator/joi**), i18n (**i18next**), docs (**swagger‑ui‑express**, **redoc**, **spectral**, **oasdiff**, **Prism/Dredd**).

Use a consistent response envelope (e.g., `{ data, meta, error }`), stable error codes, and never echo secrets.

* * *

## 1) Basic image upload with size & MIME verification

**Goal:** Accept only small images and persist safely.  
**Build steps:**

1. Add `POST /files` with `multer` memory or disk storage; limit size to **2 MB**.

2. Validate MIME by **sniffing magic bytes** (`file-type`) instead of trusting `Content‑Type`/extension; allow `image/jpeg|png|webp`.

3. Generate a **safe server filename**: `{uuid}.{ext}`, no user‑controlled paths.

4. Return `{ id, url }` for later download.  
    **Acceptance criteria:**

- Non‑image or >2 MB → `400` with `code:"UNSUPPORTED_MEDIA_TYPE"` or `PAYLOAD_TOO_LARGE`.

- Valid upload → `201` with stable URL; file stored under a dedicated uploads dir.

* * *

## 2) Enforce image dimensions & orientation

**Goal:** Reject overly large images and normalize orientation.  
**Build steps:**

1. Probe dimensions with `sharp(metadata)`; max **4096×4096**.

2. Auto‑rotate to respect EXIF; strip EXIF (privacy).

3. Save normalized image; include `{ width, height }` in response.  
    **Acceptance criteria:**

- > 4096px in either dimension → `400` with reason.

- EXIF‑rotated source returns normalized (upright) image.

* * *

## 3) Derivatives: thumbnail & medium sizes

**Goal:** Produce multiple renditions for UI usage.  
**Build steps:**

1. On upload, create `thumb 200px` and `medium 1024px` (longest side), preserving aspect ratio.

2. Store alongside original; return `_links` to renditions.  
    **Acceptance criteria:**

- Each rendition exists; dimensions never exceed targets.

- Response `_links.{thumb,medium,original}` are valid.

* * *

## 4) Streaming download with HTTP range & ETag

**Goal:** Efficient large file delivery.  
**Build steps:**

1. `GET /files/:id` serves from disk using **range requests**; support `Range` → `206 Partial Content` with `Content‑Range`.

2. Add strong ETag (hash or inode+mtime) and support `If‑None‑Match` (→ `304`) and `If‑Range`.

3. Advertise `Accept‑Ranges: bytes`.  
    **Acceptance criteria:**

- Byte‑range request returns `206` with correct headers and partial body.

- Re‑request with `If‑None‑Match` returns `304` when unchanged.

* * *

## 5) Path traversal & filename safety

**Goal:** Prevent directory escape and header abuses.  
**Build steps:**

1. Serve only by **opaque id**, not by filename path.

2. On attachment download, set `Content‑Disposition` with **RFC 5987** encoding for original filename; sanitize to avoid CR/LF.

3. Deny `..` or absolute paths anywhere.  
    **Acceptance criteria:**

- Attempts like `/files/../../etc/passwd` → `404`/`400` (no traversal).

- Non‑ASCII filenames download with correct name on major browsers.

* * *

## 6) Quarantine + antivirus scanning (async)

**Goal:** Don’t publish unscanned files.  
**Build steps:**

1. Save uploads under `/quarantine`; enqueue scan job (e.g., ClamAV via child process or container).

2. Only after **clean** result, move to `/public` and mark `status:"ready"`.

3. Expose `GET /files/:id/status`.  
    **Acceptance criteria:**

- Before scan completes, direct download → `404` or `409 PENDING`.

- EICAR test file → `422 INFECTED`; never published.

* * *

## 7) Object storage (S3) with pre‑signed upload

**Goal:** Offload transfer to the client.  
**Build steps:**

1. `POST /files/presign` returns S3 **pre‑signed POST** or `putObject` URL with content‑type and max size constraints.

2. Store metadata (owner, mime, size) after client confirms upload via callback `POST /files/complete`.

3. Serve via **signed GET** or CDN public URL depending on ACL.  
    **Acceptance criteria:**

- Uploading with wrong content‑type or oversize is rejected by S3.

- After `/complete`, `GET /files/:id` proxies or redirects to signed URL.

* * *

## 8) Resumable/chunked uploads with checksum

**Goal:** Reliable large upload flow.  
**Build steps:**

1. Create an upload session `POST /uploads` → `{ uploadId }`.

2. Accept `PUT /uploads/:uploadId/parts/:n` chunks; verify **Content‑MD5** per part.

3. `POST /uploads/:uploadId/complete` assembles parts; verify total checksum (e.g., SHA‑256).  
    **Acceptance criteria:**

- Missing/invalid checksum → `400`.

- Completing with missing parts → `409`.

- Final file hash matches the declared hash.

* * *

## 9) Retention & orphan cleanup job

**Goal:** Manage storage lifecycle.  
**Build steps:**

1. Mark files **referenced** by domain entities; others are **orphan**.

2. Nightly job removes quarantined files older than 24 h and orphans older than 7 days.

3. Expose `/admin/storage/stats` (counts, bytes by state).  
    **Acceptance criteria:**

- Orphans past retention are deleted; referenced files remain.

- Stats reflect real counts/sizes.

* * *

## 10) Input sanitization pipeline (strings)

**Goal:** Canonicalize and reject dangerous input.  
**Build steps:**

1. Create a sanitizer: trim, collapse whitespace, **Unicode NFKC normalization**, remove control chars except `\n\t`, and strip HTML tags (server‑side).

2. Apply to all user‑provided strings (query, params, body) via middleware; keep raw copy only if needed for audit.

3. Log when sanitation alters input (debug only).  
    **Acceptance criteria:**

- Control chars like `\u0000` cause `400` or are removed per policy (documented).

- HTML tags are removed; normalized output used downstream.

* * *

## 11) Central validation with Zod/Ajv + shared schemas

**Goal:** One source of truth for validation.  
**Build steps:**

1. Define schemas for `User`, `Task`, `FileMeta`, and common queries (`limit`, `offset`, `sort`).

2. Compile and use as middleware; map errors to `{ error: { code:'VALIDATION_ERROR', details:[{path,msg}] } }`.

3. Export schemas to OpenAPI components (zod‑to‑openapi or Ajv + generator).  
    **Acceptance criteria:**

- Bad bodies/queries return `400` with clear, field‑scoped details.

- Schema changes flow into docs (see exercise 13/14).

* * *

## 12) Internationalized errors & messages

**Goal:** Localize validation and server errors.  
**Build steps:**

1. Integrate `i18next` with locales `en`, `pt-BR`, `es`; parse `Accept‑Language` and set `req.locale` with fallback.

2. Put error templates in locale files; interpolate fields and limits.

3. Add `?lang=` override for testing.  
    **Acceptance criteria:**

- `Accept‑Language: pt-BR` returns Portuguese error messages.

- Unknown locale falls back to English; codes remain stable.

* * *

## 13) OpenAPI 3.0.3 spec (covering auth, tasks, files)(swagger-ui-express)

**Goal:** First‑class API contract.  
**Build steps:**

1. Create `openapi.yaml` with **servers**, **tags**, **components** (schemas, securitySchemes), global error schema.

2. Document `multipart/form-data` for `POST /files` and binary responses for downloads.

3. Add examples for success/error and auth flows; include pagination params as reusable components.  
    **Acceptance criteria:**

- Spec passes validation (no errors) via `swagger-parser` / `openapi-schema-validator`.

- `swagger-ui-express` serves docs at `/docs`; examples render correctly.

* * *

## 14) Spec quality gates: lint, CI, and drift check

**Goal:** Keep docs high‑quality and in sync.  
**Build steps:**

1. Add **Spectral** with rules (operationId, tags, 2xx/4xx examples, no ambiguous types).

2. CI job: `spectral lint`, schema validation, and **route coverage** (all Express routes present in OpenAPI).

3. Add `oasdiff` step to detect **breaking changes** between `v1` and `v2`.  
    **Acceptance criteria:**

- CI fails on missing route documentation or breaking change without a major version bump.

- Lint violations block merge until fixed.

* * *

## 15) Versioned API & deprecation policy

**Goal:** Safely ship breaking changes.  
**Build steps:**

1. Mount `v1` and `v2` routers; change task shape in `v2` (e.g., `title` → `name`).

2. Add `Deprecation: true`, `Sunset: <RFC‑date>` and `Link: <migration-guide>; rel="deprecation"` headers to **v1**.

3. Document both versions in OpenAPI (separate files or `servers`/`x‑version`).  
    **Acceptance criteria:**

- V1 responses include deprecation headers.

- V2 endpoints reflect new schema; v1 remains backward compatible until sunset date.

* * *

## 16) HATEOAS links (HAL or JSON:API style)

**Goal:** Discoverability in responses.  
**Build steps:**

1. Wrap task responses with `_links` (`self`, `update`, `delete`, `owner`) and list navigation (`first`, `prev`, `next`).

2. Document link relations in OpenAPI (as schema with `links` object) and examples.  
    **Acceptance criteria:**

- Consumers can follow `_links` to perform allowed actions.

- Pagination links reflect current `limit/offset` or `cursor`.

* * *

## 17) Health, readiness, and diagnostics

**Goal:** Operability and safe rollouts.  
**Build steps:**

1. `/livez`: process can respond. `/readyz`: check DB, cache, object storage, AV daemon, and migrations status.

2. `/diagz`: return build info (commit, version), uptime, memory, and selected config **flags only** (no secrets).

3. Fail readiness with `503` when any dependency is down; include which check failed.  
    **Acceptance criteria:**

- Stopping AV or S3 connectivity → `/readyz` returns `503` identifying the failing check.

- `/diagz` shows commit SHA and uptime; no sensitive envs.

* * *

## 18) Robust pagination & sorting validation

**Goal:** Normalize list query behavior.  
**Build steps:**

1. Validate `limit` (default 20, max 100), `offset` (>=0), `sort` (`field:asc|desc`) against an allow‑list.

2. Document ordering semantics (filter → sort → paginate).

3. Return `422` for semantically invalid combos (choose 400 vs 422 and document).  
    **Acceptance criteria:**

- `limit=1000` → clamped or `400` per policy.

- Unknown sort field → `400` with `details:[{path:'sort', msg:'unsupported field'}]`.

* * *

## 19) Download content negotiation & Content‑Disposition

**Goal:** Control inline vs attachment and filenames.  
**Build steps:**

1. `GET /files/:id?disposition=inline|attachment` and optional `filename=` param.

2. Encode filename per RFC 5987; sanitize path separators and control chars.

3. Document accepted params and defaults in OpenAPI.  
    **Acceptance criteria:**

- `?disposition=attachment&filename=olá.png` downloads with properly encoded name.

- Missing/invalid `disposition` uses default policy.

* * *

## 20) Contract tests from OpenAPI

**Goal:** Ensure implementation matches the spec.  
**Build steps:**

1. Generate tests with **Dredd** or run **Prism** in “validator” mode against your running server.

2. Include both happy‑path and failure examples (422/400/401/403).

3. Test CI step runs contract tests and fails on mismatch.  
    **Acceptance criteria:**

- Changing a response shape without updating the spec causes CI failure.

- All documented examples pass against the live server.

* * *

### General expectations (apply to all 20)

- **Security:** Never trust client MIME/extension; always sniff bytes. Block path traversal and CRLF. Don’t store EXIF when unnecessary. Use `HttpOnly`, `Secure`, `SameSite` when transporting tokens in cookies.

- **Performance:** Stream I/O; avoid buffering whole files in memory. Prefer `pipeline`/`pump`.

- **Observability:** Log request ID, file id, size, and outcomes (created, scanned, deleted), avoiding PII in logs.

- **Docs:** Keep OpenAPI authoritative; run linters and diffs in CI; provide examples for every status code you return.

- **Validation:** Centralize rules; return stable error codes and localized messages; document both.

- Add minimal **README**: how to run, env vars, and curl examples for each exercise.

- If you want, turn the acceptance criteria into **automated tests with Supertest/Jest**,
