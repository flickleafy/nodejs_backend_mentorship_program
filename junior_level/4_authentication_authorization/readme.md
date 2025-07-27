# Authentication & Authorization

Below are **20 “Authentication & Authorization” build‑style exercises** for Node.js (JavaScript) that progress from junior to near mid‑level. Use **Express** and pick one crypto stack per exercise (e.g., `bcrypt` **or** `argon2`; `jsonwebtoken` for JWT; `express-session` + `csurf` for cookie sessions). Keep responses consistent (e.g., `{ data, error, meta }`). Each item includes **step‑by‑step rubrics** and **acceptance criteria**.

* * *

## 1) User registration with hashed passwords

**Goal:** Store credentials securely and enforce unique emails.

**Build steps**

1. Create `users` table/model `{ id, email UNIQUE, password_hash, created_at }`.

2. Validate email format and `password` length (min 8) server‑side.

3. Hash password (`bcrypt` with 10–12 rounds or `argon2id`) before insert.

4. Return minimal user info (never the hash).

**Acceptance criteria**

- Duplicate email → `409`.

- New user → `201`, body contains `{ id, email }` only.

- Verifying in DB shows `password_hash` is not equal to plaintext.

* * *

## 2) Email case‑insensitive uniqueness

**Goal:** Prevent duplicate accounts due to case differences.

**Build steps**

1. Normalize emails to lowercase at registration and login.

2. Add DB constraint or unique functional index (e.g., `LOWER(email)` in PG).

**Acceptance criteria**

- Register `Test@Example.com` then `test@example.com` → second returns `409`.

- Login works regardless of submitted case.

* * *

## 3) Login with JWT (access token)

**Goal:** Provide a signed access token.

**Build steps**

1. `POST /auth/login` with `{ email, password }`.

2. Verify password; on success, sign JWT (HS256) with `{ sub: userId, email }`, `exp` ~ 15 min.

3. Return `{ accessToken }`.

**Acceptance criteria**

- Invalid credentials → `401` with generic message.

- Valid credentials → `200` and JWT decodes to correct `sub`, `exp`.

* * *

## 4) Auth middleware (Bearer) + user context

**Goal:** Protect routes and attach authenticated user.

**Build steps**

1. Implement middleware to parse `Authorization: Bearer <jwt>`, verify, and attach `req.user`.

2. Protect `/tasks*` routes; reject otherwise.

**Acceptance criteria**

- Missing/invalid token → `401`.

- Valid token → route executes with `req.user.id` set.

* * *

## 5) User‑scoped data access

**Goal:** Enforce per‑user ownership.

**Build steps**

1. Ensure all `tasks` queries filter by `user_id = req.user.id`.

2. Reject access to resources not owned by the user.

**Acceptance criteria**

- User A cannot retrieve/update/delete User B’s tasks → `404` (or `403`, pick and document).

- New tasks are created with `user_id = req.user.id`.

* * *

## 6) Refresh tokens with rotation (token family)

**Goal:** Provide session longevity without long‑lived access tokens.

**Build steps**

1. Create `refresh_tokens` store `{ id, user_id, token_hash, revoked, created_at, replaced_by_id, user_agent }`.

2. `POST /auth/refresh` accepts refresh token (cookie or body), verifies, **rotates**:

    - Revoke old record; insert new; return new access + refresh tokens.
3. On logout (`POST /auth/logout`) revoke the current refresh token.

**Acceptance criteria**

- Refresh with valid token → `200` new tokens; old token becomes unusable.

- Refreshing a **used** (rotated) token → `401` and cascades revocation of family (if implemented).

- Logout → refresh token no longer works.

* * *

## 7) RBAC: `user` vs `admin`

**Goal:** Restrict privileged operations.

**Build steps**

1. Add `role` column (`user` default, `admin`).

2. Create `requireRole('admin')` middleware.

3. Only admins can `DELETE /users/:id`.

**Acceptance criteria**

- Non‑admin deleting a user → `403`.

- Admin deleting a user → `204`.

* * *

## 8) Password policy & breach check

**Goal:** Improve password hygiene.

**Build steps**

1. Enforce policy: min length 10, require character variety (document your rules).

2. Optionally integrate a local HIBP k‑anon hash check (offline copy or mock).

3. Return policy violations as structured field errors.

**Acceptance criteria**

- Weak password → `400` with explicit reasons.

- Strong password → accepted and hashed.

* * *

## 9) Email verification flow

**Goal:** Block login until email ownership verified (configurable).

**Build steps**

1. `email_verifications` table `{ user_id, token_hash, expires_at, used_at }`.

2. On registration, issue token (store hash), send link `/auth/verify?token=...`.

3. `GET /auth/verify` marks verified; login only allowed when `email_verified=true` (toggle optional for dev).

**Acceptance criteria**

- Using an expired/used token → `400`/`410`.

- Unverified user login → `403` with `code:'EMAIL_NOT_VERIFIED'`.

- After verify → login succeeds.

* * *

## 10) Password reset flow

**Goal:** Allow secure password resets.

**Build steps**

1. `POST /auth/request-reset` accepts email (always respond 200 to avoid user enumeration).

2. Issue single‑use, short‑lived token (hash stored).

3. `POST /auth/reset` with `{ token, newPassword }`:

    - Validate token, set new hash, revoke all refresh tokens for user, audit log.

**Acceptance criteria**

- Using a consumed/expired token → `400`/`410`.

- After reset, old refresh tokens cannot refresh sessions.

- Reset endpoints do not reveal whether an email exists.

* * *

## 11) Brute‑force protection (IP + account)

**Goal:** Reduce credential‑stuffing risk.

**Build steps**

1. Track failed login attempts per IP and per account windowed over 10 min.

2. After 5 failures, lock that account for 30 min (or require CAPTCHA).

3. Return generic `401` even when locked; include `Retry-After` header.

**Acceptance criteria**

- 5th–6th bad attempts → `401` with `Retry-After`.

- Successful login resets counters.

* * *

## 12) Cookie‑based sessions + CSRF

**Goal:** Alternative to JWT using server sessions.

**Build steps**

1. Configure `express-session` with secure cookie (`HttpOnly`, `SameSite=Lax|Strict`, `Secure` in prod).

2. Protect state‑changing routes with `csurf`:

    - Issue CSRF token via safe route (e.g., `GET /csrf`).

    - Require token header on POST/PUT/PATCH/DELETE.

3. Disable JWT on these routes (choose one mode or run both behind feature flag).

**Acceptance criteria**

- Missing/invalid CSRF token on POST → `403`.

- Session cookie is `HttpOnly`, `Secure` (in prod), has `SameSite`.

* * *

## 13) Two‑Factor Auth (TOTP) + backup codes

**Goal:** Strengthen auth with a second factor.

**Build steps**

1. Use `otplib` to generate a TOTP secret and QR for the user (store encrypted/hashed).

2. Add `/auth/2fa/setup` (protected): show QR, verify first code, then enable `mfa_enabled`.

3. Modify login: after password success, if `mfa_enabled`, require `/auth/2fa/verify` before issuing tokens.

4. Generate 8 one‑time backup codes (store hashed).

**Acceptance criteria**

- Wrong TOTP → `401`, right TOTP → tokens issued.

- Backup code works once; second use rejected.

- Disable 2FA only after re‑authentication.

* * *

## 14) OAuth2/OIDC social login (Google) + account linking

**Goal:** Support external identity providers.

**Build steps**

1. Use `passport` with `passport-google-oauth20` (or `openid-client`).

2. Implement `/auth/google` and callback; on success:

    - If email exists → link provider to existing user.

    - Else create a new user with `email_verified=true`.

3. Store in `external_identities` `{ user_id, provider, provider_user_id }`.

**Acceptance criteria**

- Same Google account logs into same app user on subsequent attempts.

- If an email is already taken by password user, login links it after local re‑auth (avoid account hijack).

* * *

## 15) API keys for service‑to‑service

**Goal:** Allow programmatic access without user context.

**Build steps**

1. Generate random API keys (prefix + secret), store only **hash** + metadata (owner, scopes, last\_used\_at).

2. Accept `Authorization: ApiKey <key>`.

3. Grant limited scopes via claims resolved from the key.

**Acceptance criteria**

- Valid key → `200`; invalid → `401`.

- Key can be revoked; further requests fail.

- Key scopes restrict endpoints; unauthorized → `403`.

* * *

## 16) Scope/permission checks (claims‑based)

**Goal:** Fine‑grained authorization beyond roles.

**Build steps**

1. Extend JWT (or session) with `permissions` array (e.g., `['tasks:read','tasks:write']`).

2. Create `requirePermissions(...perms)` middleware that checks **all** specified permissions.

3. Assign permissions on login based on role and user attributes.

**Acceptance criteria**

- Lacking any required permission → `403`.

- Having required permissions → success.

- Permissions survive token refresh.

* * *

## 17) Resource‑level authorization (policy layer)

**Goal:** Centralize ownership and rule checks.

**Build steps**

1. Implement `can(user).read(task)` / `can(user).update(task)` policy helpers.

2. In controllers, fetch the resource then call the policy before acting.

3. Add tests: self‑access, cross‑user, admin override.

**Acceptance criteria**

- User cannot read/update another user’s task unless admin.

- Policy code is unit‑tested independently of routes.

* * *

## 18) Multi‑tenancy guard (tenant isolation)

**Goal:** Enforce tenant boundaries.

**Build steps**

1. Add `tenant_id` to `users` and `tasks`.

2. Inject `req.tenantId` from token (claim) or subdomain.

3. Repositories must always filter by tenant; add automated test that attempts cross‑tenant access.

**Acceptance criteria**

- Cross‑tenant read/write attempts → `404`/`403`.

- New tasks inherit `tenant_id` from the request context.

* * *

## 19) Session/device management & revoke‑one

**Goal:** List and revoke specific sessions without logging out everywhere.

**Build steps**

1. Track refresh token metadata: `{ id, user_id, created_at, ip, user_agent, last_used_at }`.

2. `GET /auth/sessions` lists active sessions for current user.

3. `DELETE /auth/sessions/:id` revokes **only** that session.

**Acceptance criteria**

- Listing shows at least `created_at`, `ip`, `user_agent`.

- Deleting one session invalidates only that refresh token; others remain valid.

* * *

## 20) Security headers & token transport hardening

**Goal:** Ship secure defaults.

**Build steps**

1. Add `helmet` and configure:

    - `Content-Security-Policy` (at least default‑src ‘self’).

    - `Referrer-Policy`, `X-Frame-Options` (or CSP frame-ancestors), `X-Content-Type-Options`, `Permissions-Policy`.

2. For JWT in cookies (optional mode): use `HttpOnly`, `Secure`, `SameSite=Lax|Strict`, short `Max-Age`.

3. CORS: allowlist origins, `credentials` only when using cookies.

4. Limit JWT size, set reasonable expirations, and rotate secrets.

**Acceptance criteria**

- `GET /healthz` shows headers in response.

- Cookies (if used) have correct flags in prod.

- Disallowed origin requests are blocked (verified via preflight and server logs).

* * *

### General expectations (apply to all 20)

- **No plaintext secrets** in logs or responses.

- **Error mapping**: invalid creds → `401`; missing auth → `401`; forbidden → `403`; conflict → `409`; validation → `400`.

- **Replay resilience**: one‑time tokens (verification/reset) are stored as **hashes** with expiry, single‑use.

- **Testability**: Provide at least one happy‑path and one failure test per new route/flow (Supertest/Jest).

- **Graceful degradation**: Feature‑flag advanced features (2FA, OAuth) for local dev.

- Add minimal **README**: how to run, env vars, and curl examples for each exercise.

- If you want, turn the acceptance criteria into **automated tests with Supertest/Jest**,
