# Persistence (SQLite or PostgreSQL via Knex/Sequelize)

Below are **20 “Persistence (SQLite or PostgreSQL via Knex/Sequelize)” build‑style exercises** for Node.js (JavaScript) with **step‑by‑step rubrics** and **acceptance criteria**. They progress from basic DB setup to the upper end of junior level (nearing mid). Choose **one** stack for each exercise:

- **Knex + (sqlite3 or pg)**, or

- **Sequelize + (sqlite3 or pg)**.

Keep routes and behavior consistent with your previous layers (health, CRUD, etc.). Prefer a clean architecture: `routes/`, `controllers/`, `repositories/`, `db/`, `migrations/`, `seeds/`.

* * *

## 1) DB connection module & pooling

**Goal:** Replace in‑memory store with a real DB connection.

**Build steps:**

1. Install drivers: `sqlite3` (or `pg`) plus `knex` **or** `sequelize`.

2. Create `db/index.js` exporting a singleton connection/pool. Load config from `.env` (`DB_CLIENT`, `DB_URL`, pool min/max).

3. Add startup probe: run `SELECT 1` (PG) or `SELECT 1` (SQLite) and log success/failure.

4. Wire graceful shutdown to close pool on `SIGTERM`/`SIGINT`.

**Acceptance criteria:**

- App boots only if the DB is reachable; otherwise exits with a clear error.

- Pool limits reflect env values.

- Shutdown closes connections cleanly (no Node “open handle” warnings).

* * *

## 2) Initial migration: `users` & `tasks` tables

**Goal:** Define normalized schema.

**Build steps:**

1. Create migration `001_init` with tables:

    - `users(id PK, email UNIQUE NOT NULL, password_hash NOT NULL, created_at, updated_at)`

    - `tasks(id PK, user_id FK users(id) NOT NULL, title NOT NULL, done DEFAULT false, created_at, updated_at)`

2. Use UTC timestamps with sensible defaults (`DEFAULT NOW()` in PG; triggers or app‑set for SQLite).

3. Add indexes: `tasks(user_id)`.

**Acceptance criteria:**

- `knex migrate:latest` or `sequelize db:migrate` creates both tables.

- Foreign key exists and is enforced.

- Unique on `users.email` works (duplicate insert fails).

* * *

## 3) DB‑backed repository layer

**Goal:** Encapsulate SQL in a repository.

**Build steps:**

1. Create `repositories/usersRepo.js` and `repositories/tasksRepo.js`.

2. Implement `create`, `findById`, `findAllByUser`, `update`, `remove` (hard delete for now).

3. Map DB rows → plain JS entities with camelCase keys.

**Acceptance criteria:**

- Controllers call only repository methods (no SQL in controllers).

- All CRUD flows work against DB (in‑memory removed).

- Unit tests (or manual curl) show parity with previous in‑memory behavior.

* * *

## 4) Seed scripts (users + realistic tasks)

**Goal:** Deterministic local data.

**Build steps:**

1. Add seeders to insert: 2 users, ~20 tasks split across users with varied `done`, `title`, and dates.

2. Ensure seeds are idempotent (truncate first or upsert by email).

3. Wire `npm run db:seed`.

**Acceptance criteria:**

- Running seeds twice doesn’t duplicate rows.

- After seeding, `GET /tasks` for each user returns expected counts.

* * *

## 5) Unique constraint: `title` per `user_id`

**Goal:** Enforce business uniqueness at DB layer.

**Build steps:**

1. New migration adding **composite unique** on `tasks(user_id, title)`.

2. Update repository to map unique‑violation errors to HTTP `409 CONFLICT`.

3. Write a negative test: insert duplicate title for the same user.

**Acceptance criteria:**

- Duplicate titles across **different** users allowed.

- Duplicate titles for the **same** user → `409` with `{ error.code:"DUPLICATE_TITLE" }`.

* * *

## 6) Migrations for nullable `due_date` + rollback

**Goal:** Safe schema evolution.

**Build steps:**

1. Migration `add_due_date_nullable` on `tasks`.

2. Provide down/rollback to drop the column.

3. Backfill script (optional) to set varied `due_date` in seeds.

4. Extend repo/validators to accept `dueDate`.

**Acceptance criteria:**

- `migrate:latest` adds column; `migrate:rollback` removes it without breaking other columns.

- `POST /tasks` accepts optional `dueDate` and persists it.

* * *

## 7) Query performance & indexing `(done, due_date)`

**Goal:** Speed up typical list queries.

**Build steps:**

1. Create a composite index on `tasks(user_id, done, due_date)`.

2. Add query timing logs around repo list methods (start/stop time).

3. Compare timings pre/post index on a dataset of at least ~5k tasks (generate via seed).

**Acceptance criteria:**

- Index exists (verified via `\d tasks` in psql or pragma in SQLite).

- Logged timings improve for `GET /tasks?done=true&sort=due_date`.

* * *

## 8) Pagination in DB (limit/offset)

**Goal:** Do pagination server‑side with total counts.

**Build steps:**

1. Implement `findPagedByUser({ limit, offset, filters, sort })` using `LIMIT/OFFSET`.

2. Run a separate `COUNT(*)` query for total (reuse filters).

3. Return `{ rows, total }` to controller.

**Acceptance criteria:**

- `GET /tasks?limit=10&offset=20` returns 10 items and `meta.total` equals the full filtered count.

- SQL shows proper binding/parameters (no string concatenation).

* * *

## 9) Keyset (cursor) pagination `(created_at, id)`

**Goal:** Page without large offsets.

**Build steps:**

1. Add repo method `findAfterCursor({ userId, cursor, limit })` using tuple `(created_at, id)` ordering.

2. Encode/decode cursor to a compact string.

3. Return `nextCursor` when more records exist.

**Acceptance criteria:**

- First page (no cursor) returns stable order; following `nextCursor` retrieves the next slice with no gaps/overlaps.

- Cannot combine `cursor` with `offset` (returns `400`).

* * *

## 10) Transactions: create task + activity

**Goal:** Atomic multi‑table write.

**Build steps:**

1. Create `task_activities(id PK, task_id FK, kind, created_at)`.

2. In route `POST /tasks`, wrap creation of `tasks` + `task_activities(kind='CREATED')` in **one transaction**.

3. On any failure, the whole tx rolls back.

**Acceptance criteria:**

- Simulated failure (e.g., invalid activity insert) leaves **no** orphan task.

- Successful create returns the task and has exactly one corresponding activity row.

* * *

## 11) Upsert (PUT semantics)

**Goal:** Replace or insert in one call.

**Build steps:**

1. Implement `PUT /tasks/:id`:

    - If exists: replace `title`, `done`, `due_date`.

    - If not: insert with the provided `id`.

2. Use **upsert**: PG `INSERT ... ON CONFLICT (id) DO UPDATE`; SQLite `INSERT ... ON CONFLICT`.

**Acceptance criteria:**

- Non‑existent id → `201` and row inserted with that id.

- Existing id → `200` and row replaced (no extra columns left from previous state).

* * *

## 12) Optimistic concurrency via `version`

**Goal:** Prevent lost updates.

**Build steps:**

1. Migration: add integer `version DEFAULT 0 NOT NULL` to `tasks`.

2. `GET /tasks/:id` returns ETag `"v<version>"`.

3. `PATCH/PUT` require `If-Match: "v<version>"` and perform `UPDATE ... WHERE id=? AND version=?; SET version=version+1`.

4. Map mismatch to `412 Precondition Failed`.

**Acceptance criteria:**

- Update without `If-Match` → `428 Precondition Required` (or `400`, document choice).

- Stale `If-Match` → `412`; correct → succeeds and increments version.

* * *

## 13) Pessimistic locking (advanced; Postgres preferred)

**Goal:** Demonstrate row locks for critical sections.

**Build steps:**

1. In a transaction, fetch the task with `SELECT ... FOR UPDATE` (PG) or note SQLite’s coarse locking limitations.

2. Simulate two concurrent updates; second should wait until first commits.

3. Enforce a max lock wait via statement timeout or app‑level timeout.

**Acceptance criteria:**

- With PG: concurrent update shows serialization behavior; logs prove the second update waited.

- With SQLite: document limitation and emulate via app mutex; tests reflect chosen strategy.

* * *

## 14) Retry on transient errors (deadlocks/timeouts)

**Goal:** Make writes resilient.

**Build steps:**

1. Wrap transactional code in a retry helper with exponential backoff (max 3 attempts) for retryable errors (deadlock, serialization failure).

2. Log attempt count and the final outcome.

**Acceptance criteria:**

- Inject a simulated retryable error on first attempt; second succeeds; client sees only one successful response.

- Non‑retryable errors are not retried and bubble up with correct HTTP status.

* * *

## 15) Check constraints & data quality at DB level

**Goal:** Enforce invariants close to data.

**Build steps:**

1. Migration: add `CHECK (length(title) BETWEEN 1 AND 200)` and (optionally PG) `CHECK (due_date IS NULL OR due_date >= '1970-01-01')`.

2. Validate also at app layer to return friendly 400s **before** DB errors when possible.

3. Map DB check violations to `400` with `code:'CONSTRAINT_VIOLATION'`.

**Acceptance criteria:**

- Too‑long title rejected at app‑layer; if it slips through, DB rejects it.

- Due date goofy value (e.g., 1800‑01‑01) → `400`/constraint error mapped.

* * *

## 16) FK actions: CASCADE vs RESTRICT

**Goal:** Understand deletion semantics.

**Build steps:**

1. Add `ON DELETE CASCADE` to `tasks.user_id` in a migration variant **A**; add `ON DELETE RESTRICT` in variant **B** (pick one and document).

2. When deleting a user:

    - CASCADE: their tasks and activities are also deleted.

    - RESTRICT: deletion blocked if tasks exist.

**Acceptance criteria:**

- Behavior matches chosen FK policy.

- Tests show either cascading removal or a prevented user delete with `409`/`400`.

* * *

## 17) Soft delete + partial unique index (PG) or scoped uniqueness

**Goal:** Keep historical rows while preserving uniqueness for active items.

**Build steps:**

1. Add `deleted_at` nullable on `tasks`.

2. Change “delete” to set `deleted_at=now()` (soft delete).

3. For PG: create a **partial unique index** on `(user_id, title) WHERE deleted_at IS NULL`.  
    For SQLite: emulate by enforcing uniqueness in app logic inside a transaction.

4. Make list endpoints exclude soft‑deleted by default; add `include_deleted` flag.

**Acceptance criteria:**

- After soft delete, a new task can re‑use the same title for that user.

- Uniqueness still enforced among non‑deleted tasks.

- `include_deleted=true` reveals soft‑deleted rows.

* * *

## 18) N+1 elimination with JOINs

**Goal:** Fetch related data efficiently.

**Build steps:**

1. Add `GET /tasks?include=user` that returns each task with `{ user: { id, email } }`.

2. Implement a **single** JOIN query (or eager load in Sequelize) rather than per‑row queries.

3. Prove with logs that only one query is executed for the list.

**Acceptance criteria:**

- Response embeds user info for each task.

- Logs show 1 query for the list path (not N queries).

* * *

## 19) Database health & migration state in `/readyz`

**Goal:** Production‑grade readiness.

**Build steps:**

1. Update `/readyz` to check:

    - DB reachable (`SELECT 1`),

    - Latest migrations applied (check `knex_migrations` or `SequelizeMeta`),

    - Optionally, table existence (`tasks`, `users`).

2. If any check fails, return `503` with details (no secrets).

**Acceptance criteria:**

- Breaking any condition (e.g., pending migrations) flips readiness to `503`.

- Healthy state returns `200` with `{ checks:{db:true,migrations:true} }`.

* * *

## 20) Audit fields and triggers/hooks

**Goal:** Keep `created_at`/`updated_at` consistent.

**Build steps:**

1. Ensure all tables have `created_at`/`updated_at` with DB defaults.

2. For PG: add a trigger function to set `updated_at = NOW()` on UPDATE; for SQLite: enforce in app layer or use `generated columns` where available.

3. Remove all manual timestamp writes from the app (except on insert if needed for SQLite).

**Acceptance criteria:**

- Updates change `updated_at` automatically (verify two consecutive updates yield later timestamps).

- Inserts set both fields; app code no longer sets `updated_at` explicitly.

* * *

### Implementation notes (apply to all)

- **Parameterization**: Never concatenate user input into SQL. Verify bound parameters in logs.

- **Error mapping**: Unique → `409`, FK violation → `409`/`400`, check violation → `400`, not found → `404`.

- **Testing**: Prefer **transactional tests** (wrap each test in a tx and roll back) when using Postgres; for SQLite, recreate schema per test suite.

- **Separation of Concerns**: Routes → Controllers → Repositories → DB. Migrations/seeds are independent.

- **Observability**: Log SQL timings at DEBUG level, but avoid logging secrets or full payloads.

- Add minimal **README**: how to run, env vars, and curl examples for each exercise.

- If you want, turn the acceptance criteria into **automated tests with Supertest/Jest**,
