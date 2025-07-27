# Data Modeling & Validation

Below are **10 build‑style exercises** for Data Modeling & Validation in Node.js with SQL databases. Each exercise includes step-by-step instructions and clear acceptance criteria that can be verified with unit or integration tests.

* * *

## 1) Define a basic table schema

**Goal:** Create a `users` table with essential fields.
**Build steps:**

1. Use Knex or Sequelize to define a migration for `users`.
2. Include `id`, `email`, `password_hash`, and `created_at` fields.
**Acceptance criteria:**

- Migration creates the table with correct columns and types.
- Unit test: Assert table exists and columns are present.

* * *

## 2) Add constraints for data integrity

**Goal:** Enforce uniqueness and non-null constraints.
**Build steps:**

1. Add `UNIQUE` constraint to `email` and `NOT NULL` to all fields except `id`.
2. Try inserting duplicate or null values.
**Acceptance criteria:**

- Duplicate email fails; null fields fail.
- Unit test: Assert constraint enforcement.

* * *

## 3) Normalize data with related tables

**Goal:** Create a normalized schema for tasks.
**Build steps:**

1. Add a `tasks` table with `id`, `user_id`, `title`, `done`, `created_at`.
2. Set `user_id` as a foreign key referencing `users(id)`.
**Acceptance criteria:**

- Foreign key constraint enforced.
- Integration test: Assert valid and invalid foreign keys.

* * *

## 4) Add default values and timestamps

**Goal:** Use sensible defaults for fields.
**Build steps:**

1. Set `done` default to `false` and `created_at` to current timestamp.
2. Insert a new task and check defaults.
**Acceptance criteria:**

- Defaults applied when fields are omitted.
- Unit test: Assert default values.

* * *

## 5) Validate input before insert

**Goal:** Prevent invalid data from entering the database.
**Build steps:**

1. Use a validation library (e.g., Joi, Zod) to validate `title` and `email` before insert.
2. Reject invalid input with a clear error.
**Acceptance criteria:**

- Invalid input is not inserted; error returned.
- Unit test: Assert validation and error handling.

* * *

## 6) Enforce referential integrity

**Goal:** Prevent orphaned records.
**Build steps:**

1. Try deleting a user with existing tasks.
2. Set up `ON DELETE CASCADE` or restrict deletion.
**Acceptance criteria:**

- Orphaned tasks are not allowed; cascade or restrict works.
- Integration test: Assert referential integrity.

* * *

## 7) Add indexes for performance

**Goal:** Speed up queries with indexes.
**Build steps:**

1. Add an index on `tasks(user_id)`.
2. Benchmark query performance before and after.
**Acceptance criteria:**

- Index exists and improves query speed.
- Unit test: Assert index presence.

* * *

## 8) Validate updates and partial changes

**Goal:** Ensure updates are valid and safe.
**Build steps:**

1. Implement PATCH for tasks, allowing partial updates.
2. Validate updated fields before saving.
**Acceptance criteria:**

- Only valid updates are accepted; invalid ones rejected.
- Unit test: Assert update validation.

* * *

## 9) Handle errors and constraint violations

**Goal:** Return clear errors for DB violations.
**Build steps:**

1. Catch DB errors (e.g., duplicate, null, foreign key).
2. Map errors to user-friendly messages and status codes.
**Acceptance criteria:**

- Errors are caught and mapped to clear responses.
- Integration test: Assert error handling for violations.

* * *

## 10) Test schema migrations and rollbacks

**Goal:** Ensure migrations work and can be rolled back.
**Build steps:**

1. Run migration to add a new field (e.g., `updated_at`).
2. Roll back the migration and verify schema.
**Acceptance criteria:**

- Migration adds/removes field as expected.
- Unit test: Assert schema before and after rollback.

* * *

### Notes for students

- Always validate input before database operations.
- Use constraints and indexes for integrity and performance.
- Test migrations, updates, and error handling thoroughly.
