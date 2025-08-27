# Interview Questions Bank

Fundamentals
- Explain REST constraints. Which ones are most often compromised in practice and why?
- REST vs SOAP vs GraphQL – trade-offs and when to choose each.
- Difference between API and Web Service.

HTTP & Design
- Map CRUD to HTTP methods; when is POST vs PUT appropriate?
- Common 2xx/4xx/5xx codes and when to return each.
- Design clean resource URLs and when to use path vs query params.

Security
- Compare session cookies, JWT, and OAuth 2.0; when to choose each.
- CORS: what is a preflight and how to secure it?
- Rate limiting strategies and headers to expose.

Reliability & Scale
- Idempotency keys: how implemented on the server side?
- Pagination patterns (cursor vs offset) and trade-offs.
- Optimistic concurrency with ETags vs version fields.

Advanced
- Caching with Cache-Control/ETag; validation vs freshness.
- HATEOAS: benefits and practical downsides.
- Async operations (202) and job-status endpoints.

Testing & Ops
- Contract testing vs integration testing; role of OpenAPI.
- Monitoring SLIs/SLOs and alerting basics for APIs.
