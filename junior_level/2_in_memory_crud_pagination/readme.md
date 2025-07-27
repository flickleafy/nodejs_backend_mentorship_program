#  In‑Memory CRUD & Pagination

Below are **20 “In‑Memory CRUD & Pagination” build‑style exercises** for Node.js (JavaScript) with **step‑by‑step rubrics** and **acceptance criteria**. They progress from basic to the upper end of junior level. Keep everything **in memory** (no database). Prefer **Express**, keep **GET endpoints side‑effect free**, and use a consistent response envelope like `{ data, meta, error }`.

* * *

## 1) Define the Task model & in‑memory store

**Goal:** Establish a clear data shape and a single source of truth in memory.

**Build steps:**

1. Define a Task shape: `{ id, title, done, createdAt, updatedAt }`.

2. Use a safe ID generator (UUID or ULID) in a small `id.js` utility.

3. Create a module `store.js` exporting an array `tasks = []` and helpers to read/write it (no direct mutation outside).

4. Seed with 3 demo tasks on startup.

**Acceptance criteria:**

- Task `id` is unique, string.

- `done` defaults to `false`; `createdAt` and `updatedAt` are ISO strings.

- `store.js` is the only module holding the array; other modules call its exported functions.

* * *

## 2) CRUD endpoints (in‑memory)

**Goal:** Implement canonical CRUD for tasks.

**Build steps:**

1. Routes: `POST /tasks`, `GET /tasks`, `GET /tasks/:id`, `PATCH /tasks/:id`, `DELETE /tasks/:id`.

2. `POST` requires `{ title }`; sets defaults.

3. `PATCH` allows partial update of `{ title?, done? }`.

4. `DELETE` removes from memory; return `204`.

**Acceptance criteria:**

- `POST /tasks` → `201` with new task.

- `GET /tasks/:id` → `200` with task or `404`.

- `PATCH` updates only provided fields; `updatedAt` changes.

- `DELETE` unknown id → `404`; existing → `204`.

* * *

## 3) Validation & defaults

**Goal:** Enforce types and required fields.

**Build steps:**

1. Validate `title` is non‑empty string (1–200 chars).

2. Validate `done` if present is boolean.

3. On validation error: `400` with `{ error: { code:'VALIDATION_ERROR', details:[...] } }`.

**Acceptance criteria:**

- Missing/empty `title` on `POST` → `400`.

- Non‑boolean `done` on `PATCH` → `400`.

- Valid input sets `done=false` by default.

* * *

## 4) Consistent response envelope

**Goal:** Standardize success/error payloads.

**Build steps:**

1. All success: `{ data: <object|array>, meta?: <object> }`.

2. All errors via centralized handler: `{ error: { message, code, requestId? } }`.

3. No stacks in responses.

**Acceptance criteria:**

- Inspect 3+ endpoints: envelope shape is consistent.

- Unknown route → `404` with `{ error:{ code:'NOT_FOUND' } }`.

* * *

## 5) Offset pagination & total count

**Goal:** List tasks with `limit/offset` and include totals.

**Build steps:**

1. `GET /tasks?limit=&offset=` with sane defaults (e.g., `limit=10`, max 100).

2. Return `{ data:[...], meta:{ total, limit, offset } }`.

3. Validate query types; clamp `limit` at max.

**Acceptance criteria:**

- `GET /tasks?limit=2&offset=1` returns 2 items (or fewer) and `meta.total`.

- Invalid `limit=-1` → `400`.

* * *

## 6) Sorting by `createdAt`

**Goal:** Allow order control.

**Build steps:**

1. Add `sort=createdAt:asc|desc` to `GET /tasks`.

2. Implement a **stable** comparator (ties keep original order).

3. Sorting occurs **after** filtering but **before** pagination (document the order).

**Acceptance criteria:**

- `sort=createdAt:asc` and `:desc` produce reversed sequences over same dataset.

- Stability verified with equal timestamps retaining relative order.

* * *

## 7) Filtering by `done`

**Goal:** Server‑side filtering.

**Build steps:**

1. Add `done=true|false` to `GET /tasks`.

2. Combine with pagination + sorting.

**Acceptance criteria:**

- `GET /tasks?done=true&limit=5` returns only completed tasks.

- Works with sorting and returns correct `meta.total` for the filtered set.

* * *

## 8) Idempotent POST with `Idempotency-Key`

**Goal:** Prevent duplicate creations on client retries.

**Build steps:**

1. On `POST /tasks`, read `Idempotency-Key`.

2. Keep an in‑memory map `{ key → createdTaskId }` with TTL (e.g., 24 h).

3. If key seen, return `200` with the **same** task; otherwise create and store.

**Acceptance criteria:**

- Two identical `POST`s with same key → second returns the same `id`, `200`.

- Different keys create distinct tasks.

* * *

## 9) Path parameter validation

**Goal:** Early reject invalid IDs.

**Build steps:**

1. Validate `:id` format (UUID/ULID) in a param middleware.

2. If invalid, return `400` with `code:'INVALID_ID'`.

**Acceptance criteria:**

- `/tasks/not-an-id` → `400`.

- A valid‑looking but non‑existent id → `404`.

* * *

## 10) PATCH semantics & unknown fields

**Goal:** Robust partial updates.

**Build steps:**

1. Allow only `{ title?, done? }`.

2. Reject unknown fields with `400`; do not silently ignore.

3. `updatedAt` must always change when any field changes.

**Acceptance criteria:**

- Sending `{ foo: 1 }` → `400`.

- No‑op PATCH (same values) → `200` but **must not** change `updatedAt` (documented behavior).

* * *

## 11) Bulk create

**Goal:** Efficiency and atomicity in memory.

**Build steps:**

1. `POST /tasks/bulk` with `{ items: [{ title, done? }, ...] }`.

2. Validate **all** items first; if any invalid → reject whole batch.

3. Limit batch size to, say, 50.

**Acceptance criteria:**

- Valid batch → `201` with `{ data:{ created:[ids...] }, meta:{ count } }`.

- Any invalid item → `400` with detailed index‑based errors; nothing created.

* * *

## 12) Bulk update `done` status

**Goal:** Batch modifications.

**Build steps:**

1. `PATCH /tasks/bulk` with `{ ids: [id...], done: boolean }`.

2. Validate ids array (unique, non‑empty).

3. Return `{ data:{ updated, notFound } }`.

**Acceptance criteria:**

- Mixed known/unknown ids → updates known, returns unknown in `notFound`.

- `done` must be boolean; otherwise `400`.

* * *

## 13) Text search on `title`

**Goal:** Simple search + pagination.

**Build steps:**

1. `GET /tasks?q=` filters tasks whose `title` contains `q` (case‑insensitive).

2. Trim and normalize `q`; empty `q` ignored.

3. Combine with `limit/offset` and `sort`.

**Acceptance criteria:**

- `?q=test` returns only titles containing “test” (case‑insensitive).

- `q` + `done=true` works together.

* * *

## 14) Cursor‑based pagination (by `createdAt,id`)

**Goal:** Avoid large offsets.

**Build steps:**

1. Support `?cursor=` and `?limit=`.

2. Cursor encodes the last seen pair `(createdAt,id)`; use lexicographic ordering: first by `createdAt`, then by `id`.

3. Response `meta.nextCursor` when more results exist. Disallow using `cursor` with `offset`.

**Acceptance criteria:**

- First page (no cursor) returns `limit` items and a `nextCursor` when applicable.

- Using returned `nextCursor` yields the next slice with no overlaps or gaps.

* * *

## 15) Multi‑field sorting

**Goal:** Flexible ordering.

**Build steps:**

1. Accept `sort=done:asc,createdAt:desc` (comma‑separated list).

2. Implement stable multi‑key comparator.

3. Validate fields and directions; unknown field → `400`.

**Acceptance criteria:**

- Data first ordered by `done`, then by `createdAt` within each `done` group.

- Invalid sort field → `400` with helpful message.

* * *

## 16) JSON Schema validation (Ajv)

**Goal:** Centralized, testable validation.

**Build steps:**

1. Define JSON Schemas for create and update payloads.

2. Use Ajv (or similar) to validate bodies in middleware.

3. Map Ajv errors to a clean `details[]` list (path, message).

**Acceptance criteria:**

- Known invalid payload returns `400` with readable field paths.

- Valid payloads bypass the schema middleware and reach the handlers.

* * *

## 17) PUT replace semantics

**Goal:** Full replacement of a resource.

**Build steps:**

1. Implement `PUT /tasks/:id` that **replaces** the task with `{ title, done }`.

2. If task exists → replace fields, update timestamps.

3. If not exists → **create** the task (upsert), preserving the id from the path (document this behavior).

**Acceptance criteria:**

- PUT with missing `title` → `400`.

- Existing id → replaced (only allowed fields exist afterwards).

- Non‑existing id → created; `201`.

* * *

## 18) Soft delete with `deletedAt`

**Goal:** Preserve records for listing variants.

**Build steps:**

1. Change `DELETE /tasks/:id` to set `deletedAt = now` instead of removing.

2. Default listings exclude soft‑deleted.

3. Add `include_deleted=true` and `only_deleted=true` to `GET /tasks`.

**Acceptance criteria:**

- After “delete”, `GET /tasks` hides the task; `GET /tasks?include_deleted=true` shows it with `deletedAt`.

- `only_deleted=true` returns only soft‑deleted tasks.

- Deleting an already soft‑deleted task remains idempotent (`204`).

* * *

## 19) Concurrency control with ETag/If‑Match

**Goal:** Prevent lost updates.

**Build steps:**

1. Maintain integer `version` per task; increase on every successful modification.

2. Expose ETag as `"v<version>"` on `GET /tasks/:id`.

3. Require `If-Match` on `PATCH`/`PUT`; mismatch → `412 Precondition Failed`.

**Acceptance criteria:**

- `PATCH` without `If-Match` → `428 Precondition Required` (or `400`, but document choice).

- With stale `If-Match` → `412`.

- With correct `If-Match` → update succeeds and ETag increments.

* * *

## 20) Aggregations & projections for listings

**Goal:** Add useful list metadata and selective fields.

**Build steps:**

1. Add `GET /tasks/stats` returning counts `{ total, done, notDone, deleted }` (soft‑deleted aware).

2. Add `fields=` query to `GET /tasks` (comma‑separated) to project properties, e.g., `fields=id,title`.

3. Validate requested fields; unknown → `400`. Always include `id` even if not requested.

**Acceptance criteria:**

- `GET /tasks/stats` reflects current in‑memory state.

- `GET /tasks?fields=title,done` returns objects with exactly `id,title,done` (and still respects filters/pagination).

- Unknown field in `fields` → `400`.

* * *

### General expectations (apply to all exercises)

- **Order of operations for listings:** (1) filter → (2) sort → (3) paginate. Document this.

- **No side effects in GET** (read‑only).

- **Input validation** for body and query. Clear error messages.

- **Consistent envelope** and HTTP status codes.

- **Separation of Concerns:** `routes/`, `middleware/validation`, `lib/sort`, `store/`.

- **Clean code:** small functions, no duplication, pure comparators for sorting, immutable read operations (no mutation during GET).

- Add minimal **README**: how to run, env vars, and curl examples for each exercise.
- If you want, turn the acceptance criteria into **automated tests with Supertest/Jest**,
