# HTTP Status Codes

HTTP status codes indicate the result of an HTTP request. Understanding proper status code usage is crucial for building intuitive APIs and effective error handling. Status codes are grouped into five classes, each serving specific purposes in REST API communication.

## Status Code Classes

### 1xx - Informational
**Purpose**: Request received, continuing process
**Usage**: Rarely used in REST APIs

| Code | Name | Description | REST Usage |
|------|------|-------------|------------|
| 100 | Continue | Server received headers, client should send body | WebSocket upgrades |
| 101 | Switching Protocols | Server switching protocols | WebSocket handshake |
| 102 | Processing | Server processing (WebDAV) | Long-running operations |

### 2xx - Success
**Purpose**: Request successfully received, understood, and accepted

| Code | Name | Description | REST Usage |
|------|------|-------------|------------|
| 200 | OK | Standard success response | GET, PUT, PATCH with data |
| 201 | Created | Resource created successfully | POST operations |
| 202 | Accepted | Request accepted for processing | Async operations |
| 204 | No Content | Success but no response body | DELETE, PUT/PATCH without data |
| 206 | Partial Content | Partial resource delivered | Range requests |

### 3xx - Redirection
**Purpose**: Further action needed to complete request

| Code | Name | Description | REST Usage |
|------|------|-------------|------------|
| 301 | Moved Permanently | Resource permanently moved | API versioning |
| 302 | Found | Resource temporarily moved | Temporary redirects |
| 304 | Not Modified | Resource not modified | Conditional requests |
| 307 | Temporary Redirect | Temporary redirect, preserve method | API maintenance |
| 308 | Permanent Redirect | Permanent redirect, preserve method | API migration |

### 4xx - Client Error
**Purpose**: Client error, bad request syntax or cannot be fulfilled

| Code | Name | Description | REST Usage |
|------|------|-------------|------------|
| 400 | Bad Request | Invalid request syntax/data | Validation errors |
| 401 | Unauthorized | Authentication required | Missing/invalid auth |
| 403 | Forbidden | Access denied | Insufficient permissions |
| 404 | Not Found | Resource not found | Invalid endpoints/IDs |
| 405 | Method Not Allowed | HTTP method not supported | Wrong method used |
| 406 | Not Acceptable | Cannot produce acceptable response | Content negotiation |
| 409 | Conflict | Request conflicts with current state | Business rule violations |
| 410 | Gone | Resource permanently deleted | Deprecated endpoints |
| 411 | Length Required | Content-Length header required | Missing content length |
| 412 | Precondition Failed | Precondition in headers failed | Conditional requests |
| 413 | Payload Too Large | Request entity too large | File upload limits |
| 415 | Unsupported Media Type | Media type not supported | Wrong content type |
| 422 | Unprocessable Entity | Semantic errors in request | Business validation |
| 429 | Too Many Requests | Rate limit exceeded | API throttling |

### 5xx - Server Error
**Purpose**: Server failed to fulfill valid request

| Code | Name | Description | REST Usage |
|------|------|-------------|------------|
| 500 | Internal Server Error | Generic server error | Unexpected errors |
| 501 | Not Implemented | Method not implemented | Unsupported features |
| 502 | Bad Gateway | Invalid response from upstream | Proxy/gateway errors |
| 503 | Service Unavailable | Server temporarily unavailable | Maintenance mode |
| 504 | Gateway Timeout | Gateway timeout | Upstream timeouts |

## Detailed Usage Examples

### 2xx Success Responses

#### 200 OK - Standard Success
```http
# GET request with data
GET /api/users/123
Response: 200 OK
Content-Type: application/json

{
  "id": 123,
  "name": "John Doe",
  "email": "john@example.com"
}

# PUT request with updated data
PUT /api/users/123
Content-Type: application/json
{
  "name": "John Smith",
  "email": "john.smith@example.com"
}

Response: 200 OK
Content-Type: application/json
{
  "id": 123,
  "name": "John Smith",
  "email": "john.smith@example.com",
  "updated_at": "2025-08-27T10:30:00Z"
}
```

#### 201 Created - Resource Creation
```http
# Create new user
POST /api/users
Content-Type: application/json

{
  "name": "Jane Doe",
  "email": "jane@example.com"
}

Response: 201 Created
Location: /api/users/124
Content-Type: application/json

{
  "id": 124,
  "name": "Jane Doe",
  "email": "jane@example.com",
  "created_at": "2025-08-27T10:45:00Z"
}
```

#### 202 Accepted - Asynchronous Processing
```http
# Start async job
POST /api/reports/generate
Content-Type: application/json

{
  "type": "sales_report",
  "period": "2025-08"
}

Response: 202 Accepted
Content-Type: application/json

{
  "job_id": "job_abc123",
  "status": "processing",
  "status_url": "/api/jobs/job_abc123",
  "estimated_completion": "2025-08-27T11:00:00Z"
}
```

#### 204 No Content - Success Without Response Data
```http
# Delete resource
DELETE /api/users/123

Response: 204 No Content

# Update without returning data
PUT /api/users/123/preferences
Content-Type: application/json

{
  "theme": "dark",
  "notifications": false
}

Response: 204 No Content
```

### 3xx Redirection Responses

#### 301 Moved Permanently - API Versioning
```http
# Old API version
GET /api/v1/users/123

Response: 301 Moved Permanently
Location: /api/v2/users/123
Content-Type: application/json

{
  "message": "API v1 is deprecated. Please use v2.",
  "new_url": "/api/v2/users/123"
}
```

#### 304 Not Modified - Conditional Requests
```http
# Request with ETag
GET /api/users/123
If-None-Match: "abc123"

Response: 304 Not Modified
ETag: "abc123"
# No response body
```

### 4xx Client Error Responses

#### 400 Bad Request - Validation Errors
```http
# Invalid request data
POST /api/users
Content-Type: application/json

{
  "name": "",
  "email": "invalid-email"
}

Response: 400 Bad Request
Content-Type: application/json

{
  "error": "validation_failed",
  "message": "Request validation failed",
  "details": [
    {
      "field": "name",
      "code": "required",
      "message": "Name is required"
    },
    {
      "field": "email",
      "code": "invalid_format",
      "message": "Email format is invalid"
    }
  ]
}
```

#### 401 Unauthorized - Authentication Required
```http
# Missing authentication
GET /api/users/123

Response: 401 Unauthorized
Content-Type: application/json
WWW-Authenticate: Bearer realm="api"

{
  "error": "authentication_required",
  "message": "Valid authentication credentials required",
  "auth_url": "/api/auth/login"
}

# Invalid token
GET /api/users/123
Authorization: Bearer invalid_token

Response: 401 Unauthorized
Content-Type: application/json

{
  "error": "invalid_token",
  "message": "The provided authentication token is invalid or expired",
  "auth_url": "/api/auth/refresh"
}
```

#### 403 Forbidden - Insufficient Permissions
```http
# Valid auth but insufficient permissions
DELETE /api/users/123
Authorization: Bearer valid_user_token

Response: 403 Forbidden
Content-Type: application/json

{
  "error": "insufficient_permissions",
  "message": "You don't have permission to delete users",
  "required_role": "admin"
}
```

#### 404 Not Found - Resource Missing
```http
# Non-existent resource
GET /api/users/999

Response: 404 Not Found
Content-Type: application/json

{
  "error": "resource_not_found",
  "message": "User with ID 999 not found",
  "resource_type": "user",
  "resource_id": "999"
}

# Non-existent endpoint
GET /api/invalid-endpoint

Response: 404 Not Found
Content-Type: application/json

{
  "error": "endpoint_not_found",
  "message": "The requested endpoint does not exist",
  "available_endpoints": "/api/docs"
}
```

#### 405 Method Not Allowed
```http
# Wrong HTTP method
POST /api/users/123

Response: 405 Method Not Allowed
Allow: GET, PUT, PATCH, DELETE
Content-Type: application/json

{
  "error": "method_not_allowed",
  "message": "POST method is not allowed for this endpoint",
  "allowed_methods": ["GET", "PUT", "PATCH", "DELETE"]
}
```

#### 409 Conflict - Business Rule Violations
```http
# Duplicate resource
POST /api/users
Content-Type: application/json

{
  "name": "John Doe",
  "email": "existing@example.com"
}

Response: 409 Conflict
Content-Type: application/json

{
  "error": "resource_conflict",
  "message": "User with this email already exists",
  "conflicting_field": "email",
  "existing_resource": "/api/users/123"
}

# State conflict
DELETE /api/orders/456
Authorization: Bearer token

Response: 409 Conflict
Content-Type: application/json

{
  "error": "invalid_state",
  "message": "Cannot delete order that has been shipped",
  "current_state": "shipped",
  "allowed_states": ["pending", "cancelled"]
}
```

#### 422 Unprocessable Entity - Semantic Errors
```http
# Valid format but business rule violations
POST /api/events
Content-Type: application/json

{
  "title": "Meeting",
  "start_date": "2025-08-27T10:00:00Z",
  "end_date": "2025-08-27T09:00:00Z"
}

Response: 422 Unprocessable Entity
Content-Type: application/json

{
  "error": "business_rule_violation",
  "message": "End date cannot be before start date",
  "violations": [
    {
      "rule": "date_range_validation",
      "message": "End date must be after start date",
      "fields": ["start_date", "end_date"]
    }
  ]
}
```

#### 429 Too Many Requests - Rate Limiting
```http
# Rate limit exceeded
GET /api/users
Authorization: Bearer token

Response: 429 Too Many Requests
Retry-After: 60
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 0
X-RateLimit-Reset: 1693123200
Content-Type: application/json

{
  "error": "rate_limit_exceeded",
  "message": "API rate limit exceeded",
  "limit": 1000,
  "window": "1 hour",
  "retry_after": 60
}
```

### 5xx Server Error Responses

#### 500 Internal Server Error
```http
# Unexpected server error
GET /api/users/123

Response: 500 Internal Server Error
Content-Type: application/json

{
  "error": "internal_server_error",
  "message": "An unexpected error occurred",
  "request_id": "req_abc123",
  "timestamp": "2025-08-27T10:30:00Z"
}
```

#### 503 Service Unavailable - Maintenance
```http
# Service maintenance
GET /api/users

Response: 503 Service Unavailable
Retry-After: 3600
Content-Type: application/json

{
  "error": "service_unavailable",
  "message": "Service temporarily unavailable for maintenance",
  "maintenance_end": "2025-08-27T12:00:00Z",
  "status_page": "https://status.example.com"
}
```

## Status Code Selection Guidelines

### Resource Operations
```http
# Creating resources
POST /api/users → 201 Created (with Location header)
POST /api/users/123/posts → 201 Created

# Reading resources
GET /api/users/123 → 200 OK (resource found)
GET /api/users/999 → 404 Not Found (resource missing)
GET /api/users → 200 OK (collection, even if empty)

# Updating resources
PUT /api/users/123 → 200 OK (updated, returning data)
PUT /api/users/123 → 204 No Content (updated, no data returned)
PATCH /api/users/123 → 200 OK (partial update with data)

# Deleting resources
DELETE /api/users/123 → 204 No Content (deleted successfully)
DELETE /api/users/999 → 404 Not Found (resource didn't exist)
```

### Error Scenarios
```http
# Authentication issues
No auth header → 401 Unauthorized
Invalid token → 401 Unauthorized
Expired token → 401 Unauthorized

# Authorization issues
Valid auth, wrong permissions → 403 Forbidden
Admin-only endpoint accessed by user → 403 Forbidden

# Request issues
Invalid JSON → 400 Bad Request
Missing required fields → 400 Bad Request
Valid format, business rule violation → 422 Unprocessable Entity

# Resource conflicts
Creating duplicate → 409 Conflict
State transition not allowed → 409 Conflict
```

## Common Mistakes to Avoid

### 1. Using Wrong Success Codes
```http
# Wrong
POST /api/users → 200 OK

# Correct
POST /api/users → 201 Created
```

### 2. Returning 200 for Errors
```http
# Wrong
POST /api/users
Response: 200 OK
{
  "success": false,
  "error": "Validation failed"
}

# Correct
POST /api/users
Response: 400 Bad Request
{
  "error": "validation_failed",
  "message": "Request validation failed"
}
```

### 3. Confusing 401 vs 403
```http
# 401 - Authentication problem
Missing or invalid credentials

# 403 - Authorization problem
Valid credentials but insufficient permissions
```

### 4. Using 404 for Business Logic Errors
```http
# Wrong
POST /api/orders/123/ship
Response: 404 Not Found (when order can't be shipped)

# Correct
POST /api/orders/123/ship
Response: 422 Unprocessable Entity (business rule violation)
```

## Error Response Format Best Practices

### Consistent Error Structure
```json
{
  "error": "error_code",
  "message": "Human-readable message",
  "details": [],
  "request_id": "req_123",
  "timestamp": "2025-08-27T10:30:00Z",
  "documentation_url": "https://docs.api.com/errors/error_code"
}
```

### Validation Error Details
```json
{
  "error": "validation_failed",
  "message": "Request validation failed",
  "details": [
    {
      "field": "email",
      "code": "invalid_format",
      "message": "Email must be a valid email address",
      "rejected_value": "invalid-email"
    }
  ]
}
```

## Interview Questions

1. **Q: When should you use 201 vs 200 for successful requests?**
   **A:** Use 201 Created when a new resource is successfully created (typically POST operations). Use 200 OK for successful operations that don't create new resources (GET, PUT, PATCH) or when returning the created resource data.

2. **Q: What's the difference between 401 and 403 status codes?**
   **A:** 401 Unauthorized indicates authentication is required or failed (missing/invalid credentials). 403 Forbidden means the request is authenticated but the user lacks permission to access the resource.

3. **Q: When should you use 422 instead of 400?**
   **A:** Use 400 Bad Request for syntax errors, malformed JSON, or invalid request format. Use 422 Unprocessable Entity for semantically correct requests that violate business rules or constraints.

4. **Q: How do you handle partial failures in bulk operations?**
   **A:** Return 207 Multi-Status with individual status codes for each item, or use 200/422 with a response body detailing success/failure for each item in the batch.

5. **Q: What status code should be returned for successful DELETE operations?**
   **A:** Use 204 No Content for successful deletions when not returning data, or 200 OK if returning information about the deleted resource. Never use 404 if the resource was successfully deleted.

6. **Q: How should you handle rate limiting in APIs?**
   **A:** Return 429 Too Many Requests with Retry-After header indicating when the client can retry. Include rate limit headers (X-RateLimit-Limit, X-RateLimit-Remaining, X-RateLimit-Reset).

7. **Q: What's the purpose of 3xx status codes in REST APIs?**
   **A:** Indicate redirection is needed. Common uses: 301/308 for API versioning, 304 for conditional requests (caching), 302/307 for temporary redirects during maintenance.

8. **Q: How do you indicate that a long-running operation has started?**
   **A:** Use 202 Accepted to indicate the request was accepted for processing. Include a job ID and status URL where clients can check progress.

9. **Q: What status code should you use for validation errors?**
   **A:** Use 400 Bad Request for format/syntax validation errors. Use 422 Unprocessable Entity for business rule validation errors. Include detailed error information in the response body.

10. **Q: How should 5xx errors be handled in production APIs?**
    **A:** Log detailed error information server-side but return generic error messages to clients for security. Include request IDs for tracing. Use 500 for unexpected errors, 503 for maintenance, 502/504 for gateway issues.
