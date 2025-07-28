# Advanced Environment Management & Configuration

Below are **10 build‑style exercises** for Advanced Environment Management & Configuration in Node.js. Each exercise includes step-by-step instructions and clear acceptance criteria that can be verified with unit or integration tests.

* * *

## 1) Use environment variables for configuration

**Goal:** Store sensitive and environment-specific values outside code.
**Build steps:**

1. Use `dotenv` to load variables from a `.env` file.
2. Access variables in your app (e.g., `process.env.DB_URL`).
**Acceptance criteria:**

- App reads config from environment variables.
- Unit test: Assert config values are loaded from `.env`.

* * *

## 2) Validate required environment variables at startup

**Goal:** Fail fast if required config is missing.
**Build steps:**

1. Check for required variables (e.g., `PORT`, `DB_URL`) at startup.
2. Exit with a clear error if any are missing.
**Acceptance criteria:**

- App exits with error if required vars are missing.
- Unit test: Assert error on missing config.

* * *

## 3) Support multiple environment profiles

**Goal:** Use different configs for dev, test, and prod.
**Build steps:**

1. Use `NODE_ENV` to load different `.env` files (e.g., `.env.development`).
2. Document how to switch profiles.
**Acceptance criteria:**

- App loads correct config for each environment.
- Unit test: Assert profile switching and config loading.

* * *

## 4) Use hierarchical configuration

**Goal:** Organize config in a structured way.
**Build steps:**

1. Use a config library (e.g., `config`, `convict`) for nested config.
2. Store config in files by environment.
**Acceptance criteria:**

- Config is organized and overrides work as expected.
- Unit test: Assert config structure and overrides.

* * *

## 5) Secure secrets with environment injection

**Goal:** Keep secrets out of code and images.
**Build steps:**

1. Remove secrets from code and `.env` in production.
2. Inject secrets at runtime via environment or mounted files.
**Acceptance criteria:**

- No secrets are present in code or images.
- Integration test: Assert secrets are injected at runtime.

* * *

## 6) Rotate secrets and credentials

**Goal:** Change secrets without downtime.
**Build steps:**

1. Use a secrets manager or rotate secrets manually.
2. Reload secrets in the app without restart.
**Acceptance criteria:**

- Secrets can be rotated and reloaded.
- Integration test: Assert rotation and reload.

* * *

## 7) Parameterize Docker and Compose configs

**Goal:** Use environment variables in Docker and Compose files.
**Build steps:**

1. Reference env vars in `Dockerfile` and `docker-compose.yml`.
2. Document required variables for deployment.
**Acceptance criteria:**

- Docker and Compose use env vars for config.
- Unit test: Assert parameterization in configs.

* * *

## 8) Validate config at runtime

**Goal:** Ensure all config values are valid.
**Build steps:**

1. Use a schema (e.g., Joi) to validate config on startup.
2. Fail with a clear error if invalid.
**Acceptance criteria:**

- Invalid config is detected and rejected.
- Unit test: Assert validation and error handling.

* * *

## 9) Document all configuration options

**Goal:** Make config easy to understand and manage.
**Build steps:**

1. List all config options in README or a separate file.
2. Explain purpose, type, and default for each.
**Acceptance criteria:**

- All config options are documented and explained.
- Unit test: Assert documentation matches code.

* * *

## 10) Test configuration in CI/CD pipelines

**Goal:** Ensure config works in automated deployments.
**Build steps:**

1. Set up CI/CD to inject and validate config.
2. Run tests with different config profiles.
**Acceptance criteria:**

- CI/CD runs with correct config and passes tests.
- Integration test: Assert config in pipeline runs.

* * *

### Notes for students

- Always validate and document configuration.
- Use environment variables and secrets managers for sensitive data.
- Test config in all environments and deployment pipelines.
