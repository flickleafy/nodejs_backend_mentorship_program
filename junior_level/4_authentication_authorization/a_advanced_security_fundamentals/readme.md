# Advanced Security Fundamentals

Below are **10 build‑style exercises** for Advanced Security Fundamentals in Node.js. Each exercise includes step-by-step instructions and clear acceptance criteria that can be verified with unit or integration tests.

* * *

## 1) Enforce strong password policies

**Goal:** Require secure passwords during registration.
**Build steps:**

1. Validate passwords for minimum length (8+), mixed case, number, and symbol.
2. Reject weak passwords with a clear error.
**Acceptance criteria:**

- Weak passwords are rejected with status 400 and error message.
- Unit test: Assert validation for various password inputs.

* * *

## 2) Hash passwords securely

**Goal:** Store passwords using a strong hash algorithm.
**Build steps:**

1. Use `bcrypt` or `argon2` to hash passwords before saving.
2. Never store plaintext passwords.
**Acceptance criteria:**

- Passwords are hashed and not stored in plaintext.
- Unit test: Assert hash presence and uniqueness.

* * *

## 3) Prevent duplicate accounts (case-insensitive email)

**Goal:** Enforce unique emails regardless of case.
**Build steps:**

1. Normalize emails to lowercase before saving.
2. Add a unique constraint to the email field.
**Acceptance criteria:**

- Duplicate emails (case-insensitive) are rejected with status 409.
- Unit test: Assert uniqueness enforcement.

* * *

## 4) Implement JWT authentication with expiration

**Goal:** Use signed tokens for user sessions.
**Build steps:**

1. Issue JWTs on login with a short expiration (e.g., 15 min).
2. Validate JWTs on protected routes.
**Acceptance criteria:**

- Valid JWT grants access; expired/invalid JWT returns 401.
- Integration test: Assert token issuance and validation.

* * *

## 5) Protect against brute-force login attacks

**Goal:** Limit failed login attempts.
**Build steps:**

1. Track failed login attempts per user/IP.
2. Lock account or delay response after several failures.
**Acceptance criteria:**

- Excessive failures result in lockout or delay.
- Unit test: Assert lockout after threshold.

* * *

## 6) Secure session cookies

**Goal:** Use secure, HTTP-only cookies for sessions.
**Build steps:**

1. Set cookies with `Secure`, `HttpOnly`, and `SameSite` attributes.
2. Test cookie behavior in browser and API.
**Acceptance criteria:**

- Cookies are set with all security attributes.
- Integration test: Assert cookie flags and access restrictions.

* * *

## 7) Validate and sanitize user input

**Goal:** Prevent injection attacks.
**Build steps:**

1. Use a validation library to check all user input.
2. Sanitize input to remove dangerous characters.
**Acceptance criteria:**

- Malicious input is rejected or sanitized.
- Unit test: Assert validation and sanitization.

* * *

## 8) Implement CSRF protection

**Goal:** Prevent cross-site request forgery.
**Build steps:**

1. Use a CSRF protection library (e.g., `csurf`).
2. Require CSRF tokens on state-changing requests.
**Acceptance criteria:**

- Requests without valid CSRF token are rejected with 403.
- Integration test: Assert CSRF enforcement.

* * *

## 9) Scan dependencies for vulnerabilities

**Goal:** Detect and fix insecure packages.
**Build steps:**

1. Use a tool like `npm audit` or `snyk` to scan dependencies.
2. Fix or update any vulnerable packages.
**Acceptance criteria:**

- Vulnerabilities are detected and resolved.
- Unit test: Assert scan results and remediation.

* * *

## 10) Log and monitor security events

**Goal:** Track authentication and authorization events.
**Build steps:**

1. Log all login attempts, password changes, and failed access.
2. Integrate logs with a monitoring tool.
**Acceptance criteria:**

- Security events are logged and visible in monitoring.
- Integration test: Assert log entries for key events.

* * *

### Notes for students

- Always validate and sanitize input.
- Use secure password storage and session management.
- Monitor and log security events.
- Scan dependencies regularly for vulnerabilities.
