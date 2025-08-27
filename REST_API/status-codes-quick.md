# Status Codes Quick Reference

2xx Success
- 200 OK – Standard success
- 201 Created – New resource created; include Location
- 202 Accepted – Async processing; return status URL
- 204 No Content – Success without body

3xx Redirection
- 301/308 – Permanent redirect
- 302/307 – Temporary redirect
- 304 – Not Modified (ETag/If-None-Match)

4xx Client Errors
- 400 – Bad Request (validation)
- 401 – Unauthorized (no/invalid auth)
- 403 – Forbidden (insufficient permissions)
- 404 – Not Found
- 405 – Method Not Allowed
- 409 – Conflict (state/version)
- 415 – Unsupported Media Type
- 422 – Unprocessable Entity (semantic errors)
- 429 – Too Many Requests

5xx Server Errors
- 500 – Internal Server Error
- 502 – Bad Gateway
- 503 – Service Unavailable
- 504 – Gateway Timeout
