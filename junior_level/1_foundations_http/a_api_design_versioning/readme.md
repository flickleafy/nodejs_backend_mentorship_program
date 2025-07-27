# API Design & Versioning

Below are **10 build‑style exercises** for API Design & Versioning in Node.js. Each exercise includes step-by-step instructions and clear acceptance criteria that can be verified with unit or integration tests. These will help you understand RESTful principles, versioning strategies, and how to build maintainable APIs.

* * *

## 1) Create a RESTful GET endpoint for a resource

**Goal:** Build a simple GET endpoint for `/books`.
**Build steps:**

1. Set up an Express app and add a route `GET /books`.
2. Return a static array of book objects as JSON.

**Acceptance criteria:**

- `GET /books` returns status 200 and a JSON array of books.
- Response shape matches `{ data: [...] }`.
- Unit test: Assert status, shape, and content.

* * *

## 2) Implement a POST endpoint with validation

**Goal:** Accept new books via POST.
**Build steps:**

1. Add `POST /books` accepting `{ title, author }` in the body.
2. Validate both fields are non-empty strings.
3. On success, return the created book with status 201.
4. On validation error, return 400 and error details.

**Acceptance criteria:**

- Valid body returns 201 and the new book.
- Invalid body returns 400 and error message.
- Unit test: Test both valid and invalid cases.

* * *

## 3) Add API versioning via URL prefix

**Goal:** Support `/v1/books` and `/v2/books` endpoints.
**Build steps:**

1. Refactor routes to use `/v1/books`.
2. Add `/v2/books` that returns an extra field (e.g., `publishedYear`).

**Acceptance criteria:**

- `GET /v1/books` returns books without `publishedYear`.
- `GET /v2/books` returns books with `publishedYear`.
- Integration test: Assert both endpoints and their shapes.

* * *

## 4) Handle unknown routes and methods

**Goal:** Return clear errors for unsupported paths/methods.
**Build steps:**

1. Add a catch-all 404 handler for unknown endpoints.
2. For known routes, reject unsupported methods with 405 and `Allow` header.

**Acceptance criteria:**

- Unknown route returns 404 and JSON error.
- Unsupported method returns 405 and `Allow` header.
- Integration test: Assert both cases.

* * *

## 5) Use proper HTTP status codes for CRUD

**Goal:** Communicate results with correct status codes.
**Build steps:**

1. Implement `GET`, `POST`, `PUT`, and `DELETE` for `/books`.
2. Return 200 for success, 201 for creation, 204 for deletion, 400 for bad request, 404 for not found.

**Acceptance criteria:**

- Each method returns the correct status code.
- Unit test: Assert status codes for all CRUD actions.

* * *

## 6) Standardize response envelope

**Goal:** Use a consistent response shape for all endpoints.
**Build steps:**

1. Wrap all responses in `{ data, error, meta }`.
2. Ensure errors use `{ error: { message, code } }`.

**Acceptance criteria:**

- All endpoints return the envelope.
- Error responses include code and message.
- Unit test: Assert envelope for success and error.

* * *

## 7) Document endpoints with OpenAPI (Swagger)

**Goal:** Write a Swagger spec for your API.
**Build steps:**

1. Create a `swagger.yaml` or use Swagger UI.
2. Document `GET /books` and `POST /books` with parameters, responses, and examples.

**Acceptance criteria:**

- Spec is valid and covers both endpoints.
- Integration test: Validate spec with Swagger tools.

* * *

## 8) Support content negotiation

**Goal:** Allow clients to request JSON or plain text.
**Build steps:**

1. For `GET /books`, check the `Accept` header.
2. Return JSON for `application/json`, plain text for `text/plain`.

**Acceptance criteria:**

- `Accept: application/json` returns JSON.
- `Accept: text/plain` returns a text list of book titles.
- Integration test: Assert both formats.

* * *

## 9) Add a deprecation warning for old versions

**Goal:** Warn clients about deprecated endpoints.
**Build steps:**

1. For `/v1/books`, add a `Deprecation` header in the response.
2. Document migration steps to `/v2/books`.

**Acceptance criteria:**

- `GET /v1/books` includes `Deprecation` header.
- Integration test: Assert header presence and value.

* * *

## 10) Ensure backward compatibility in changes

**Goal:** Make safe, additive changes to your API.
**Build steps:**

1. Add a new optional field to the book object (e.g., `genre`).
2. Do not remove or rename existing fields.
3. Document the change in your API spec.

**Acceptance criteria:**

- Old clients still work with the new response.
- Unit test: Assert old and new clients parse the response correctly.

* * *

### Notes for students

- Test every endpoint and error case.
- Use explicit status codes and clear error messages.
- Document your API and versioning strategy.
- Prefer additive changes for backward compatibility.
