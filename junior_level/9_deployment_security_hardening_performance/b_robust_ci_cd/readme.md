# Robust Continuous Integration/Continuous Deployment (CI/CD)

Below are **10 build‑style exercises** for Robust CI/CD in Node.js. Each exercise includes step-by-step instructions and clear acceptance criteria that can be verified with unit or integration tests.

* * *

## 1) Set up a basic CI workflow with GitHub Actions

**Goal:** Automate tests on every push.
**Build steps:**

1. Create `.github/workflows/ci.yml` for Node.js.
2. Run `npm install` and `npm test` on push and pull request.
**Acceptance criteria:**

- CI runs and passes for all pushes/PRs.
- Unit test: Assert workflow file presence and test execution.

* * *

## 2) Add code linting and formatting checks

**Goal:** Enforce code quality in CI.
**Build steps:**

1. Add ESLint and Prettier steps to CI workflow.
2. Fail CI if lint or format checks fail.
**Acceptance criteria:**

- CI fails on lint/format errors.
- Unit test: Assert lint/format checks in workflow.

* * *

## 3) Integrate security scanning in CI

**Goal:** Detect vulnerabilities automatically.
**Build steps:**

1. Add `npm audit` or Snyk scan to CI workflow.
2. Fail CI on high severity vulnerabilities.
**Acceptance criteria:**

- CI fails on detected vulnerabilities.
- Unit test: Assert security scan step and failure on issues.

* * *

## 4) Run tests in multiple Node.js versions

**Goal:** Ensure compatibility across Node.js versions.
**Build steps:**

1. Configure matrix strategy in CI for Node.js 16, 18, and 20.
2. Run tests in all versions.
**Acceptance criteria:**

- Tests pass in all configured Node.js versions.
- Integration test: Assert matrix setup and test results.

* * *

## 5) Build and test Docker images in CI

**Goal:** Validate container builds and health.
**Build steps:**

1. Add Docker build step to CI workflow.
2. Run container and test health endpoint.
**Acceptance criteria:**

- Docker image builds and passes health check.
- Integration test: Assert build and health test.

* * *

## 6) Deploy to staging environment on successful CI

**Goal:** Automate deployment to a test environment.
**Build steps:**

1. Add deployment step to CI workflow for staging.
2. Use environment secrets for deployment.
**Acceptance criteria:**

- Successful CI triggers deployment to staging.
- Integration test: Assert deployment step and result.

* * *

## 7) Run performance tests in CI

**Goal:** Check for regressions in speed.
**Build steps:**

1. Add a performance test step (e.g., autocannon) to CI.
2. Fail CI if performance drops below threshold.
**Acceptance criteria:**

- CI fails on performance regression.
- Unit test: Assert performance test and threshold enforcement.

* * *

## 8) Implement zero-downtime deployment strategy

**Goal:** Deploy updates without service interruption.
**Build steps:**

1. Use blue-green or rolling deployment in staging.
2. Document deployment strategy in README.
**Acceptance criteria:**

- Deployments do not interrupt service.
- Integration test: Assert zero-downtime during deployment.

* * *

## 9) Add rollback mechanism for failed deployments

**Goal:** Restore previous version on failure.
**Build steps:**

1. Configure CI/CD to rollback on failed deployment.
2. Test rollback by simulating a failure.
**Acceptance criteria:**

- Failed deployments trigger rollback to last good version.
- Integration test: Assert rollback works as expected.

* * *

## 10) Document CI/CD pipeline and environment variables

**Goal:** Make pipeline easy to understand and maintain.
**Build steps:**

1. Document all CI/CD steps, secrets, and required environment variables in README or a separate file.
2. Update docs with every pipeline change.
**Acceptance criteria:**

- Documentation is complete and matches pipeline.
- Unit test: Assert documentation accuracy and completeness.

* * *

### Notes for students

- Always test, lint, and scan code in CI.
- Automate deployments and document your pipeline.
- Use rollback and zero-downtime strategies for reliability.
