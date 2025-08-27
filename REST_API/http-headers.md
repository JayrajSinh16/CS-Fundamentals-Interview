# HTTP Headers in REST APIs

HTTP headers provide essential metadata about requests and responses in REST APIs. They control caching, authentication, content negotiation, CORS, and many other aspects of HTTP communication. Understanding proper header usage is crucial for building robust, secure, and efficient APIs.

## Header Categories

### Request Headers
Headers sent by clients to provide information about the request or client preferences.

### Response Headers
Headers sent by servers to provide information about the response or server capabilities.

### Representation Headers
Headers that describe the format and encoding of the message body.

### Authentication Headers
Headers used for API authentication and authorization.

## Essential HTTP Headers

### Content-Type
**Purpose**: Specifies the media type of the request/response body
**Type**: Representation header

```http
# JSON requests
POST /api/users
Content-Type: application/json

{
  "name": "John Doe",
  "email": "john@example.com"
}

# Form data
POST /api/upload
Content-Type: multipart/form-data; boundary=----formdata123

# XML requests
POST /api/data
Content-Type: application/xml

<?xml version="1.0"?>
<user>
  <name>John Doe</name>
  <email>john@example.com</email>
</user>

# URL-encoded form
POST /api/login
Content-Type: application/x-www-form-urlencoded

username=john&password=secret123
```

### Accept
**Purpose**: Specifies acceptable response media types
**Type**: Request header

```http
# Prefer JSON
GET /api/users/123
Accept: application/json

# Accept multiple formats with preferences
GET /api/users/123
Accept: application/json, application/xml;q=0.8, text/plain;q=0.5

# Accept any type
GET /api/users/123
Accept: */*

# API versioning through Accept header
GET /api/users/123
Accept: application/vnd.myapi.v2+json
```

### Authorization
**Purpose**: Contains credentials for authenticating the client
**Type**: Authentication header

```http
# Bearer token (JWT, OAuth2)
GET /api/users/123
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

# Basic authentication
GET /api/users/123
Authorization: Basic dXNlcm5hbWU6cGFzc3dvcmQ=

# API key
GET /api/users/123
Authorization: ApiKey your-api-key-here

# Custom scheme
GET /api/users/123
Authorization: Custom token=abc123,signature=xyz789
```

### Content-Length
**Purpose**: Indicates the size of the request/response body in bytes
**Type**: Representation header

```http
POST /api/users
Content-Type: application/json
Content-Length: 85

{
  "name": "John Doe",
  "email": "john@example.com",
  "role": "developer"
}
```

### User-Agent
**Purpose**: Identifies the client application
**Type**: Request header

```http
GET /api/users
User-Agent: MyApp/1.0 (iOS 15.0; iPhone13,1)

GET /api/data
User-Agent: curl/7.68.0

GET /api/status
User-Agent: PostmanRuntime/7.29.0
```

## Caching Headers

### Cache-Control
**Purpose**: Directives for caching mechanisms
**Type**: Response header

```http
# No caching for sensitive data
GET /api/users/123/profile
Response:
Cache-Control: no-cache, no-store, must-revalidate
Pragma: no-cache
Expires: 0

# Cache for 1 hour
GET /api/public/articles
Response:
Cache-Control: public, max-age=3600

# Private cache only, revalidate
GET /api/users/123/dashboard
Response:
Cache-Control: private, max-age=300, must-revalidate

# Conditional caching
GET /api/data
Response:
Cache-Control: public, max-age=0, must-revalidate
ETag: "abc123"
Last-Modified: Wed, 27 Aug 2025 10:00:00 GMT
```

### ETag (Entity Tag)
**Purpose**: Identifier for specific version of a resource
**Type**: Response header

```http
# Server response with ETag
GET /api/users/123
Response: 200 OK
ETag: "abc123def456"
Content-Type: application/json

{
  "id": 123,
  "name": "John Doe",
  "updated_at": "2025-08-27T10:00:00Z"
}

# Client conditional request
GET /api/users/123
If-None-Match: "abc123def456"

Response: 304 Not Modified
ETag: "abc123def456"

# Update with ETag validation
PUT /api/users/123
If-Match: "abc123def456"
Content-Type: application/json

{
  "name": "John Smith"
}

# If ETag doesn't match (resource was modified)
Response: 412 Precondition Failed
```

### Last-Modified
**Purpose**: Date and time when resource was last modified
**Type**: Response header

```http
# Server response
GET /api/documents/456
Response: 200 OK
Last-Modified: Wed, 27 Aug 2025 10:00:00 GMT
Content-Type: application/json

# Conditional request
GET /api/documents/456
If-Modified-Since: Wed, 27 Aug 2025 10:00:00 GMT

Response: 304 Not Modified
Last-Modified: Wed, 27 Aug 2025 10:00:00 GMT
```

## CORS Headers

### Access-Control-Allow-Origin
**Purpose**: Specifies which origins can access the resource
**Type**: Response header

```http
# Allow specific origin
OPTIONS /api/users
Response:
Access-Control-Allow-Origin: https://myapp.com

# Allow any origin (not recommended for production)
Response:
Access-Control-Allow-Origin: *

# Dynamic origin based on request
Request:
Origin: https://trusted-app.com

Response:
Access-Control-Allow-Origin: https://trusted-app.com
Vary: Origin
```

### Access-Control-Allow-Methods
**Purpose**: Specifies allowed HTTP methods for CORS requests
**Type**: Response header

```http
OPTIONS /api/users/123
Response:
Access-Control-Allow-Methods: GET, PUT, PATCH, DELETE, OPTIONS
Access-Control-Allow-Headers: Content-Type, Authorization
Access-Control-Max-Age: 86400
```

### Access-Control-Allow-Headers
**Purpose**: Specifies allowed headers in CORS requests
**Type**: Response header

```http
# Preflight request
OPTIONS /api/users
Origin: https://myapp.com
Access-Control-Request-Method: POST
Access-Control-Request-Headers: Content-Type, Authorization, X-API-Key

# Preflight response
Response: 200 OK
Access-Control-Allow-Origin: https://myapp.com
Access-Control-Allow-Methods: GET, POST, PUT, DELETE
Access-Control-Allow-Headers: Content-Type, Authorization, X-API-Key
Access-Control-Max-Age: 3600
```

## Content Negotiation Headers

### Accept-Language
**Purpose**: Preferred languages for response
**Type**: Request header

```http
# Prefer English, accept Spanish as fallback
GET /api/content/article-123
Accept-Language: en-US,en;q=0.9,es;q=0.8

Response: 200 OK
Content-Language: en-US
Content-Type: application/json

{
  "title": "Welcome to our API",
  "content": "This is an English article..."
}
```

### Accept-Encoding
**Purpose**: Acceptable content encodings (compression)
**Type**: Request header

```http
# Client supports gzip and deflate compression
GET /api/large-dataset
Accept-Encoding: gzip, deflate, br

Response: 200 OK
Content-Encoding: gzip
Content-Type: application/json
# Response body is gzip compressed
```

### Content-Encoding
**Purpose**: Encoding applied to response body
**Type**: Response header

```http
GET /api/data
Accept-Encoding: gzip

Response: 200 OK
Content-Encoding: gzip
Content-Type: application/json
Content-Length: 1234
# Compressed JSON response
```

## Custom API Headers

### Rate Limiting Headers
```http
GET /api/users
Authorization: Bearer token123

Response: 200 OK
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 999
X-RateLimit-Reset: 1693123200
X-RateLimit-Window: 3600

# When rate limit is exceeded
Response: 429 Too Many Requests
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 0
X-RateLimit-Reset: 1693123200
Retry-After: 3600
```

### API Versioning Headers
```http
# Version in custom header
GET /api/users
X-API-Version: 2.1

# Version in Accept header
GET /api/users
Accept: application/vnd.myapi.v2+json

# Response indicates version used
Response: 200 OK
X-API-Version: 2.1
Content-Type: application/vnd.myapi.v2+json
```

### Request Tracking Headers
```http
# Client sends request ID
POST /api/users
X-Request-ID: req_abc123456
Content-Type: application/json

# Server echoes request ID
Response: 201 Created
X-Request-ID: req_abc123456
Location: /api/users/789
```

### Pagination Headers
```http
GET /api/users?page=2&limit=10

Response: 200 OK
X-Total-Count: 150
X-Page: 2
X-Per-Page: 10
X-Total-Pages: 15
Link: <https://api.example.com/users?page=1&limit=10>; rel="first",
      <https://api.example.com/users?page=1&limit=10>; rel="prev",
      <https://api.example.com/users?page=3&limit=10>; rel="next",
      <https://api.example.com/users?page=15&limit=10>; rel="last"
```

## Security Headers

### Strict-Transport-Security (HSTS)
```http
Response:
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
```

### X-Content-Type-Options
```http
Response:
X-Content-Type-Options: nosniff
```

### X-Frame-Options
```http
Response:
X-Frame-Options: DENY
```

### Content-Security-Policy
```http
Response:
Content-Security-Policy: default-src 'self'; script-src 'self' 'unsafe-inline'
```

## Conditional Request Headers

### If-Match / If-None-Match
**Purpose**: Conditional requests based on ETag
**Type**: Request header

```http
# Only update if ETag matches (preventing lost updates)
PUT /api/users/123
If-Match: "abc123"
Content-Type: application/json

{
  "name": "Updated Name"
}

# Only return if ETag doesn't match (caching)
GET /api/users/123
If-None-Match: "abc123"

Response: 304 Not Modified (if ETag matches)
```

### If-Modified-Since / If-Unmodified-Since
**Purpose**: Conditional requests based on modification date
**Type**: Request header

```http
# Only return if modified since date
GET /api/documents/456
If-Modified-Since: Wed, 27 Aug 2025 10:00:00 GMT

# Only update if not modified since date
PUT /api/documents/456
If-Unmodified-Since: Wed, 27 Aug 2025 10:00:00 GMT
```

## Location Header
**Purpose**: Indicates the URL of a created or moved resource
**Type**: Response header

```http
# Resource creation
POST /api/users
Content-Type: application/json

{
  "name": "John Doe",
  "email": "john@example.com"
}

Response: 201 Created
Location: /api/users/123
Content-Type: application/json

{
  "id": 123,
  "name": "John Doe",
  "email": "john@example.com"
}

# Resource moved
GET /api/v1/users/123

Response: 301 Moved Permanently
Location: /api/v2/users/123
```

## Header Best Practices

### 1. Use Standard Headers When Possible
```http
# Good - Standard headers
Content-Type: application/json
Authorization: Bearer token
Accept: application/json

# Avoid - Custom headers for standard purposes
X-Content-Format: json
X-Auth-Token: token
X-Response-Type: json
```

### 2. Consistent Custom Header Naming
```http
# Good - Consistent X- prefix for custom headers
X-API-Version: 2.1
X-Request-ID: req_123
X-Rate-Limit: 1000

# Good - Vendor prefix
MyCompany-API-Version: 2.1
MyCompany-Request-ID: req_123
```

### 3. Proper Content-Type Usage
```http
# Specific content types
Content-Type: application/json; charset=utf-8
Content-Type: application/xml; charset=utf-8
Content-Type: text/csv; charset=utf-8

# API versioning in content type
Content-Type: application/vnd.myapi.v1+json
Content-Type: application/vnd.myapi.user.v2+json
```

### 4. Security Headers
```http
# Always include security headers
Strict-Transport-Security: max-age=31536000
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
X-XSS-Protection: 1; mode=block
```

## Common Header Mistakes

### 1. Incorrect Content-Type
```http
# Wrong - Sending JSON with wrong content type
POST /api/users
Content-Type: text/plain

{"name": "John"}

# Correct
POST /api/users
Content-Type: application/json

{"name": "John"}
```

### 2. Missing Authorization Header
```http
# Wrong - Sensitive operation without auth
DELETE /api/users/123

# Correct
DELETE /api/users/123
Authorization: Bearer valid_token
```

### 3. Ignoring Accept Header
```http
# Client request
GET /api/users/123
Accept: application/xml

# Wrong - Server ignores Accept header
Response: 200 OK
Content-Type: application/json

# Correct - Honor client preference or return 406
Response: 200 OK
Content-Type: application/xml
```

### 4. Inconsistent Header Naming
```http
# Wrong - Inconsistent naming
X-API-Version: 1.0
api-request-id: req_123
RateLimit-Remaining: 999

# Correct - Consistent naming
X-API-Version: 1.0
X-Request-ID: req_123
X-RateLimit-Remaining: 999
```

## Header Validation Examples

### Request Header Validation
```http
# Missing required Content-Type
POST /api/users
Authorization: Bearer token

{"name": "John"}

Response: 400 Bad Request
{
  "error": "missing_content_type",
  "message": "Content-Type header is required for POST requests"
}

# Unsupported Content-Type
POST /api/users
Content-Type: application/yaml

name: John

Response: 415 Unsupported Media Type
{
  "error": "unsupported_media_type",
  "message": "Content-Type 'application/yaml' is not supported",
  "supported_types": ["application/json", "application/xml"]
}
```

### Accept Header Negotiation
```http
# Client requests unsupported format
GET /api/users/123
Accept: application/yaml

Response: 406 Not Acceptable
{
  "error": "not_acceptable",
  "message": "Requested format not available",
  "available_formats": ["application/json", "application/xml"]
}
```

## Interview Questions

1. **Q: What's the difference between Content-Type and Accept headers?**
   **A:** Content-Type specifies the media type of the request/response body being sent. Accept specifies what media types the client can accept in the response. Content-Type describes what you're sending; Accept describes what you want to receive.

2. **Q: How do you implement proper API versioning using headers?**
   **A:** Use Accept header with vendor-specific media types (Accept: application/vnd.myapi.v2+json) or custom headers (X-API-Version: 2.0). The Accept header approach is more RESTful as it's part of content negotiation.

3. **Q: Explain how ETag headers work for caching and concurrency control.**
   **A:** ETags are unique identifiers for resource versions. For caching: client sends If-None-Match with ETag, server returns 304 if unchanged. For concurrency: client sends If-Match with ETag for updates, server returns 412 if ETag doesn't match (resource was modified).

4. **Q: What CORS headers are needed for a cross-origin API request?**
   **A:** Access-Control-Allow-Origin (specifies allowed origins), Access-Control-Allow-Methods (allowed HTTP methods), Access-Control-Allow-Headers (allowed request headers), and optionally Access-Control-Max-Age (preflight cache duration).

5. **Q: How should rate limiting information be communicated through headers?**
   **A:** Use X-RateLimit-Limit (total limit), X-RateLimit-Remaining (remaining requests), X-RateLimit-Reset (reset time), and Retry-After (when to retry after 429 response).

6. **Q: What's the purpose of the Location header?**
   **A:** Indicates the URL of a newly created resource (201 Created), where a resource has moved (3xx redirects), or where to find a related resource. Essential for RESTful resource creation and redirection.

7. **Q: How do you handle content negotiation in REST APIs?**
   **A:** Use Accept header for response format, Accept-Language for language preference, Accept-Encoding for compression. Server should honor preferences or return 406 Not Acceptable if unable to satisfy the request.

8. **Q: What security headers should always be included in API responses?**
   **A:** Strict-Transport-Security (HTTPS enforcement), X-Content-Type-Options: nosniff (prevent MIME sniffing), X-Frame-Options (prevent clickjacking), and Content-Security-Policy (XSS protection).

9. **Q: How do conditional requests work with If-Modified-Since?**
   **A:** Client sends If-Modified-Since header with last known modification date. Server compares with resource's Last-Modified date. If resource hasn't changed, server returns 304 Not Modified instead of full response.

10. **Q: What's the difference between Authorization and Authentication headers?**
    **A:** Authorization header contains credentials for authenticating the request (like Bearer tokens). There's no standard "Authentication" header - Authorization is the standard header for sending authentication credentials to the server.
