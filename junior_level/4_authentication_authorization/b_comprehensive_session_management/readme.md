# Comprehensive Session Management

Below are **10 build‑style exercises** for Comprehensive Session Management in Node.js. Each exercise includes step-by-step instructions and clear acceptance criteria that can be verified with unit or integration tests.

* * *

## 1) Implement basic session storage with cookies

**Goal:** Store user sessions using cookies.
**Build steps:**

1. Use `express-session` to set up session management.
2. Store session ID in a secure, HTTP-only cookie.
**Acceptance criteria:**

- Session cookie is set on login and persists across requests.
- Unit test: Assert cookie presence and session persistence.

* * *

## 2) Configure session expiration and renewal

**Goal:** Expire sessions after inactivity and renew on activity.
**Build steps:**

1. Set session timeout (e.g., 15 min).
2. Renew session expiration on each request.
**Acceptance criteria:**

- Session expires after inactivity; renews on activity.
- Unit test: Assert expiration and renewal behavior.

* * *

## 3) Store sessions in a persistent backend (Redis)

**Goal:** Use Redis for scalable session storage.
**Build steps:**

1. Configure `connect-redis` with `express-session`.
2. Store and retrieve sessions from Redis.
**Acceptance criteria:**

- Sessions persist in Redis and survive server restarts.
- Integration test: Assert session persistence and retrieval.

* * *

## 4) Secure session cookies

**Goal:** Protect session cookies from theft.
**Build steps:**

1. Set `Secure`, `HttpOnly`, and `SameSite` attributes on cookies.
2. Test cookie behavior in browser and API.
**Acceptance criteria:**

- Cookies have all security attributes.
- Unit test: Assert cookie flags and access restrictions.

* * *

## 5) Prevent session fixation attacks

**Goal:** Regenerate session ID on login.
**Build steps:**

1. Call `req.session.regenerate()` after successful login.
2. Ensure old session ID is invalidated.
**Acceptance criteria:**

- Session ID changes on login; old ID is invalid.
- Unit test: Assert session regeneration.

* * *

## 6) Implement multi-factor authentication (MFA)

**Goal:** Add a second authentication step to sessions.
**Build steps:**

1. Require a one-time code (e.g., email or app) after password login.
2. Store MFA status in session.
**Acceptance criteria:**

- MFA required before session is fully authenticated.
- Integration test: Assert MFA flow and session status.

* * *

## 7) Invalidate sessions on logout and password change

**Goal:** Remove sessions when users log out or change passwords.
**Build steps:**

1. Destroy session on logout (`req.session.destroy()`).
2. Invalidate all sessions for a user on password change.
**Acceptance criteria:**

- Sessions are destroyed and cannot be reused.
- Unit test: Assert session invalidation.

* * *

## 8) Limit concurrent sessions per user

**Goal:** Restrict users to a set number of active sessions.
**Build steps:**

1. Track active sessions per user in Redis or DB.
2. Reject new sessions if limit is reached.
**Acceptance criteria:**

- Users cannot exceed session limit.
- Integration test: Assert enforcement of session limit.

* * *

## 9) Monitor and log session events

**Goal:** Track session creation, expiration, and invalidation.
**Build steps:**

1. Log session events to a file or monitoring system.
2. Review logs for suspicious activity.
**Acceptance criteria:**

- All session events are logged and reviewable.
- Unit test: Assert log entries for session events.

* * *

## 10) Test session management under concurrent load

**Goal:** Ensure session management is robust under load.
**Build steps:**

1. Write integration tests simulating many users logging in/out.
2. Assert sessions are created, renewed, and invalidated correctly.
**Acceptance criteria:**

- Session management works under concurrent access.
- Integration test: Simulate concurrency and verify session handling.

* * *

### Notes for students

- Always secure session cookies and storage.
- Test session expiration, renewal, and invalidation.
- Use MFA and limit concurrent sessions for added security.
- Monitor and log session events for visibility.
