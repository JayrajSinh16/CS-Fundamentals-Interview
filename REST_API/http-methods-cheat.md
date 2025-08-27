# HTTP Methods Cheat Sheet

- GET: Read resource(s). Safe, idempotent. Cacheable.
- HEAD: Headers only for a GET. Safe, idempotent. Cacheable.
- OPTIONS: Capabilities for a resource (CORS preflight). Safe, idempotent.
- POST: Create or server-processed action. Not idempotent. Use 201 with Location.
- PUT: Replace entire resource at a known URI. Idempotent.
- PATCH: Partial update. Not guaranteed idempotent; design for idempotency when possible.
- DELETE: Remove resource. Idempotent (multiple deletes yield the same end state).

Typical responses
- 200 OK: GET/PUT/PATCH/DELETE success (body optional)
- 201 Created: POST success with Location header
- 204 No Content: Success without body (PUT/PATCH/DELETE)
- 400 Bad Request: Validation errors
- 401/403: Authn/Authz issues
- 404 Not Found: Missing resource
- 409 Conflict: Versioning/state conflicts
- 415/422: Media type or semantic errors

Idempotency
- GET, HEAD, OPTIONS, PUT, DELETE are idempotent by spec
- POST is not; use idempotency keys when needed (e.g., payments)
