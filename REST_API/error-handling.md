# Error Handling & Response Formats

Proper error handling is crucial for building robust and user-friendly REST APIs. Well-designed error responses help developers understand what went wrong, how to fix it, and how to prevent similar issues. Consistent error formats and meaningful error messages greatly improve the developer experience and API adoption.

## Error Response Principles

### 1. Use Appropriate HTTP Status Codes
Each error should use the most specific and appropriate HTTP status code.

```http
# Validation error
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
      "message": "Email must be a valid email address"
    }
  ]
}

# Resource not found
GET /api/users/999

Response: 404 Not Found
Content-Type: application/json
{
  "error": "resource_not_found",
  "message": "User with ID 999 not found",
  "resource_type": "user",
  "resource_id": "999"
}

# Authorization error
DELETE /api/admin/users/123
Authorization: Bearer regular_user_token

Response: 403 Forbidden
Content-Type: application/json
{
  "error": "insufficient_permissions",
  "message": "Admin role required to delete users",
  "required_permission": "admin:users:delete",
  "current_permissions": ["user:profile:read", "user:posts:write"]
}
```

### 2. Provide Consistent Error Structure
Use a standardized error response format across your entire API.

```json
{
  "error": "error_code",
  "message": "Human-readable error message",
  "details": [],
  "timestamp": "2025-08-27T10:30:00Z",
  "request_id": "req_abc123",
  "documentation_url": "https://docs.api.com/errors/error_code"
}
```

### 3. Include Actionable Information
Help developers understand how to fix or prevent the error.

```http
# Rate limit exceeded
GET /api/users
Authorization: Bearer token

Response: 429 Too Many Requests
Retry-After: 60
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 0
X-RateLimit-Reset: 1693126800
Content-Type: application/json

{
  "error": "rate_limit_exceeded",
  "message": "API rate limit exceeded",
  "details": {
    "limit": 1000,
    "window": "1 hour",
    "retry_after_seconds": 60,
    "upgrade_info": {
      "message": "Upgrade to premium for higher limits",
      "url": "https://example.com/pricing"
    }
  },
  "documentation_url": "https://docs.api.com/rate-limiting"
}
```

## Common Error Types and Responses

### Validation Errors (400 Bad Request)
```http
# Single field validation error
POST /api/users
Content-Type: application/json
{
  "email": "invalid-email-format"
}

Response: 400 Bad Request
Content-Type: application/json
{
  "error": "validation_failed",
  "message": "Request validation failed",
  "details": [
    {
      "field": "email",
      "code": "invalid_format",
      "message": "Email must be a valid email address",
      "rejected_value": "invalid-email-format",
      "expected_format": "email"
    }
  ],
  "request_id": "req_abc123"
}

# Multiple validation errors
POST /api/products
Content-Type: application/json
{
  "name": "",
  "price": -10,
  "category_id": 999,
  "tags": []
}

Response: 400 Bad Request
Content-Type: application/json
{
  "error": "validation_failed",
  "message": "Multiple validation errors occurred",
  "details": [
    {
      "field": "name",
      "code": "required",
      "message": "Product name is required"
    },
    {
      "field": "price",
      "code": "min_value",
      "message": "Price must be greater than 0",
      "rejected_value": -10,
      "min_value": 0
    },
    {
      "field": "category_id",
      "code": "not_found",
      "message": "Category with ID 999 does not exist",
      "rejected_value": 999
    },
    {
      "field": "tags",
      "code": "min_length",
      "message": "At least one tag is required",
      "min_length": 1
    }
  ],
  "request_id": "req_def456"
}

# Nested object validation
POST /api/orders
Content-Type: application/json
{
  "customer": {
    "email": "invalid",
    "address": {
      "street": "",
      "zip": "12345-invalid"
    }
  },
  "items": [
    {
      "product_id": "",
      "quantity": 0
    }
  ]
}

Response: 400 Bad Request
Content-Type: application/json
{
  "error": "validation_failed",
  "message": "Nested validation errors",
  "details": [
    {
      "field": "customer.email",
      "code": "invalid_format",
      "message": "Invalid email format"
    },
    {
      "field": "customer.address.street",
      "code": "required",
      "message": "Street address is required"
    },
    {
      "field": "customer.address.zip",
      "code": "invalid_format",
      "message": "Invalid ZIP code format",
      "expected_pattern": "^\\d{5}(-\\d{4})?$"
    },
    {
      "field": "items[0].product_id",
      "code": "required",
      "message": "Product ID is required"
    },
    {
      "field": "items[0].quantity",
      "code": "min_value",
      "message": "Quantity must be at least 1",
      "min_value": 1
    }
  ]
}
```

### Authentication Errors (401 Unauthorized)
```http
# Missing authentication
GET /api/profile

Response: 401 Unauthorized
WWW-Authenticate: Bearer realm="api"
Content-Type: application/json
{
  "error": "authentication_required",
  "message": "Authentication is required to access this resource",
  "auth_methods": ["Bearer Token"],
  "login_url": "/api/auth/login",
  "documentation_url": "https://docs.api.com/authentication"
}

# Invalid token
GET /api/profile
Authorization: Bearer invalid_token

Response: 401 Unauthorized
WWW-Authenticate: Bearer error="invalid_token", error_description="Token signature is invalid"
Content-Type: application/json
{
  "error": "invalid_token",
  "message": "The provided authentication token is invalid",
  "details": {
    "reason": "invalid_signature",
    "token_expired": false
  },
  "refresh_url": "/api/auth/refresh"
}

# Expired token
GET /api/profile
Authorization: Bearer expired_token

Response: 401 Unauthorized
WWW-Authenticate: Bearer error="invalid_token", error_description="Token has expired"
Content-Type: application/json
{
  "error": "token_expired",
  "message": "Authentication token has expired",
  "details": {
    "expired_at": "2025-08-27T10:00:00Z",
    "current_time": "2025-08-27T10:30:00Z"
  },
  "refresh_url": "/api/auth/refresh"
}
```

### Authorization Errors (403 Forbidden)
```http
# Insufficient permissions
POST /api/admin/users
Authorization: Bearer valid_user_token

Response: 403 Forbidden
Content-Type: application/json
{
  "error": "insufficient_permissions",
  "message": "You don't have permission to create users",
  "required_permissions": ["admin:users:create"],
  "current_permissions": ["user:profile:read", "user:posts:write"],
  "contact_admin": "admin@example.com"
}

# Resource ownership error
DELETE /api/posts/456
Authorization: Bearer user_token

Response: 403 Forbidden
Content-Type: application/json
{
  "error": "resource_access_denied",
  "message": "You can only delete your own posts",
  "resource_owner": "user_789",
  "current_user": "user_123"
}

# Account status error
GET /api/premium-content
Authorization: Bearer basic_user_token

Response: 403 Forbidden
Content-Type: application/json
{
  "error": "subscription_required",
  "message": "Premium subscription required to access this content",
  "current_plan": "basic",
  "required_plan": "premium",
  "upgrade_url": "https://example.com/upgrade"
}
```

### Resource Not Found (404 Not Found)
```http
# Resource doesn't exist
GET /api/users/999

Response: 404 Not Found
Content-Type: application/json
{
  "error": "resource_not_found",
  "message": "User with ID 999 not found",
  "resource_type": "user",
  "resource_id": "999",
  "suggestions": [
    "Check if the user ID is correct",
    "Verify the user hasn't been deleted",
    "Ensure you have permission to access this user"
  ]
}

# Nested resource not found
GET /api/users/123/posts/999

Response: 404 Not Found
Content-Type: application/json
{
  "error": "nested_resource_not_found",
  "message": "Post with ID 999 not found for user 123",
  "parent_resource": {
    "type": "user",
    "id": "123",
    "exists": true
  },
  "child_resource": {
    "type": "post",
    "id": "999",
    "exists": false
  }
}

# Endpoint not found
GET /api/nonexistent-endpoint

Response: 404 Not Found
Content-Type: application/json
{
  "error": "endpoint_not_found",
  "message": "The requested endpoint does not exist",
  "path": "/api/nonexistent-endpoint",
  "method": "GET",
  "available_endpoints": "/api/docs",
  "suggestions": [
    "Check the URL spelling",
    "Verify the API version",
    "Consult the API documentation"
  ]
}
```

### Method Not Allowed (405 Method Not Allowed)
```http
# Wrong HTTP method
POST /api/users/123

Response: 405 Method Not Allowed
Allow: GET, PUT, PATCH, DELETE
Content-Type: application/json
{
  "error": "method_not_allowed",
  "message": "POST method is not allowed for individual user resources",
  "method": "POST",
  "resource": "/api/users/123",
  "allowed_methods": ["GET", "PUT", "PATCH", "DELETE"],
  "suggestions": [
    "Use POST /api/users to create a new user",
    "Use PUT /api/users/123 to update the entire user",
    "Use PATCH /api/users/123 to update specific fields"
  ]
}
```

### Conflict Errors (409 Conflict)
```http
# Resource already exists
POST /api/users
Content-Type: application/json
{
  "email": "existing@example.com",
  "name": "John Doe"
}

Response: 409 Conflict
Content-Type: application/json
{
  "error": "resource_conflict",
  "message": "User with this email already exists",
  "conflicting_field": "email",
  "conflicting_value": "existing@example.com",
  "existing_resource": {
    "id": 456,
    "url": "/api/users/456"
  },
  "resolution": "Use a different email address or update the existing user"
}

# State conflict
POST /api/orders/123/ship
Authorization: Bearer token

Response: 409 Conflict
Content-Type: application/json
{
  "error": "invalid_state_transition",
  "message": "Cannot ship an order that has been cancelled",
  "current_state": "cancelled",
  "requested_action": "ship",
  "valid_transitions": ["pending -> processing", "processing -> shipped"],
  "order_id": 123
}
```

### Business Logic Errors (422 Unprocessable Entity)
```http
# Valid format but business rule violation
POST /api/events
Content-Type: application/json
{
  "title": "Meeting",
  "start_date": "2025-08-27T14:00:00Z",
  "end_date": "2025-08-27T13:00:00Z",
  "room_id": 101
}

Response: 422 Unprocessable Entity
Content-Type: application/json
{
  "error": "business_rule_violation",
  "message": "Event violates business rules",
  "violations": [
    {
      "rule": "end_after_start",
      "message": "End time must be after start time",
      "fields": ["start_date", "end_date"],
      "start_date": "2025-08-27T14:00:00Z",
      "end_date": "2025-08-27T13:00:00Z"
    },
    {
      "rule": "room_availability",
      "message": "Room 101 is not available during this time",
      "conflicting_event": {
        "id": 789,
        "title": "Board Meeting",
        "time": "2025-08-27T13:30:00Z - 2025-08-27T15:30:00Z"
      }
    }
  ]
}
```

### Server Errors (5xx)
```http
# Internal server error
GET /api/users/123

Response: 500 Internal Server Error
Content-Type: application/json
{
  "error": "internal_server_error",
  "message": "An unexpected error occurred while processing your request",
  "request_id": "req_abc123",
  "timestamp": "2025-08-27T10:30:00Z",
  "contact_support": "support@example.com",
  "status_page": "https://status.example.com"
}

# Service unavailable
GET /api/users

Response: 503 Service Unavailable
Retry-After: 300
Content-Type: application/json
{
  "error": "service_unavailable",
  "message": "Service temporarily unavailable for maintenance",
  "details": {
    "maintenance_start": "2025-08-27T10:00:00Z",
    "estimated_end": "2025-08-27T12:00:00Z",
    "reason": "Database maintenance"
  },
  "status_page": "https://status.example.com",
  "alternative_endpoints": []
}

# Gateway timeout
GET /api/external-data

Response: 504 Gateway Timeout
Content-Type: application/json
{
  "error": "gateway_timeout",
  "message": "Request to external service timed out",
  "details": {
    "upstream_service": "external-api.example.com",
    "timeout_seconds": 30,
    "retry_recommended": true
  },
  "request_id": "req_def456"
}
```

## Error Response Formats

### Standard Error Format
```json
{
  "error": "error_code",
  "message": "Human-readable error message",
  "details": {},
  "timestamp": "2025-08-27T10:30:00Z",
  "request_id": "req_abc123",
  "documentation_url": "https://docs.api.com/errors/error_code"
}
```

### RFC 7807 Problem Details Format
```http
Response: 400 Bad Request
Content-Type: application/problem+json

{
  "type": "https://example.com/probs/validation-error",
  "title": "Request validation failed",
  "status": 400,
  "detail": "One or more fields failed validation",
  "instance": "/api/users",
  "invalid-params": [
    {
      "name": "email",
      "reason": "Invalid email format"
    },
    {
      "name": "age",
      "reason": "Must be 18 or older"
    }
  ]
}
```

### JSON:API Error Format
```http
Response: 422 Unprocessable Entity
Content-Type: application/vnd.api+json

{
  "errors": [
    {
      "id": "error_1",
      "status": "422",
      "code": "validation_failed",
      "title": "Invalid email format",
      "detail": "The email address 'invalid-email' is not valid",
      "source": {
        "pointer": "/data/attributes/email"
      },
      "meta": {
        "timestamp": "2025-08-27T10:30:00Z"
      }
    },
    {
      "id": "error_2", 
      "status": "422",
      "code": "validation_failed",
      "title": "Name is required",
      "detail": "The name field cannot be empty",
      "source": {
        "pointer": "/data/attributes/name"
      }
    }
  ]
}
```

### Nested Error Structure
```json
{
  "error": "validation_failed",
  "message": "Request validation failed",
  "errors": {
    "user": {
      "email": [
        {
          "code": "invalid_format",
          "message": "Invalid email format"
        }
      ],
      "password": [
        {
          "code": "too_short",
          "message": "Password must be at least 8 characters"
        },
        {
          "code": "missing_uppercase",
          "message": "Password must contain at least one uppercase letter"
        }
      ]
    },
    "profile": {
      "birth_date": [
        {
          "code": "future_date",
          "message": "Birth date cannot be in the future"
        }
      ]
    }
  }
}
```

## Error Handling Strategies

### Progressive Error Disclosure
```http
# Basic error for general audience
Response: 400 Bad Request
{
  "error": "validation_failed",
  "message": "Please check your input and try again"
}

# Detailed error for developers (with debug header)
Request:
X-Debug-Mode: true

Response: 400 Bad Request
{
  "error": "validation_failed", 
  "message": "Request validation failed",
  "details": [
    {
      "field": "email",
      "code": "invalid_format",
      "message": "Email format is invalid",
      "regex_pattern": "^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}$",
      "rejected_value": "invalid-email"
    }
  ],
  "debug_info": {
    "validation_library": "joi v17.4.0",
    "validation_time_ms": 15,
    "stack_trace": "ValidationError: email validation failed..."
  }
}
```

### Internationalized Error Messages
```http
# Request with language preference
GET /api/users/999
Accept-Language: es-ES

Response: 404 Not Found
Content-Language: es-ES
{
  "error": "resource_not_found",
  "message": "Usuario con ID 999 no encontrado",
  "message_en": "User with ID 999 not found",
  "resource_type": "user",
  "resource_id": "999"
}

# Request with unsupported language, fallback to English
GET /api/users/999
Accept-Language: zh-CN

Response: 404 Not Found
Content-Language: en-US
{
  "error": "resource_not_found",
  "message": "User with ID 999 not found",
  "resource_type": "user", 
  "resource_id": "999",
  "note": "Requested language not available, showing English"
}
```

### Contextual Error Information
```http
# Error with context about current operation
POST /api/transfers
Content-Type: application/json
{
  "from_account": "12345",
  "to_account": "67890", 
  "amount": 1000000
}

Response: 422 Unprocessable Entity
{
  "error": "insufficient_funds",
  "message": "Transfer amount exceeds available balance",
  "details": {
    "requested_amount": 1000000,
    "available_balance": 50000,
    "currency": "USD",
    "account_id": "12345"
  },
  "suggestions": [
    "Reduce the transfer amount",
    "Add funds to your account",
    "Split the transfer into multiple smaller amounts"
  ],
  "related_endpoints": {
    "check_balance": "/api/accounts/12345/balance",
    "add_funds": "/api/accounts/12345/deposits"
  }
}
```

## Client Error Handling

### Error Response Processing
```javascript
class ApiClient {
  async request(url, options = {}) {
    try {
      const response = await fetch(url, options);
      
      if (!response.ok) {
        await this.handleErrorResponse(response);
      }
      
      return await response.json();
    } catch (error) {
      throw this.handleNetworkError(error);
    }
  }

  async handleErrorResponse(response) {
    const errorData = await response.json();
    
    switch (response.status) {
      case 400:
        throw new ValidationError(errorData);
      case 401:
        throw new AuthenticationError(errorData);
      case 403:
        throw new AuthorizationError(errorData);
      case 404:
        throw new NotFoundError(errorData);
      case 409:
        throw new ConflictError(errorData);
      case 422:
        throw new BusinessRuleError(errorData);
      case 429:
        throw new RateLimitError(errorData);
      case 500:
        throw new ServerError(errorData);
      case 503:
        throw new ServiceUnavailableError(errorData);
      default:
        throw new ApiError(errorData, response.status);
    }
  }

  handleNetworkError(error) {
    return new NetworkError({
      message: "Network request failed",
      original_error: error.message,
      timestamp: new Date().toISOString()
    });
  }
}

// Custom error classes
class ValidationError extends Error {
  constructor(errorData) {
    super(errorData.message);
    this.name = 'ValidationError';
    this.code = errorData.error;
    this.details = errorData.details;
    this.fields = this.extractFieldErrors(errorData.details);
  }

  extractFieldErrors(details) {
    const fieldErrors = {};
    if (Array.isArray(details)) {
      details.forEach(detail => {
        if (detail.field) {
          fieldErrors[detail.field] = detail.message;
        }
      });
    }
    return fieldErrors;
  }
}

class RateLimitError extends Error {
  constructor(errorData) {
    super(errorData.message);
    this.name = 'RateLimitError';
    this.retryAfter = errorData.details?.retry_after_seconds;
    this.limit = errorData.details?.limit;
  }
}
```

### Retry Logic with Exponential Backoff
```javascript
class RetryableApiClient {
  async requestWithRetry(url, options = {}, maxRetries = 3) {
    let lastError;
    
    for (let attempt = 0; attempt <= maxRetries; attempt++) {
      try {
        return await this.request(url, options);
      } catch (error) {
        lastError = error;
        
        if (!this.shouldRetry(error, attempt, maxRetries)) {
          throw error;
        }
        
        const delay = this.calculateDelay(attempt, error);
        await this.sleep(delay);
      }
    }
    
    throw lastError;
  }

  shouldRetry(error, attempt, maxRetries) {
    // Don't retry if max attempts reached
    if (attempt >= maxRetries) return false;
    
    // Don't retry client errors (4xx) except 429
    if (error instanceof ValidationError) return false;
    if (error instanceof AuthenticationError) return false;
    if (error instanceof AuthorizationError) return false;
    if (error instanceof NotFoundError) return false;
    
    // Retry server errors (5xx) and rate limits (429)
    if (error instanceof ServerError) return true;
    if (error instanceof ServiceUnavailableError) return true;
    if (error instanceof RateLimitError) return true;
    if (error instanceof NetworkError) return true;
    
    return false;
  }

  calculateDelay(attempt, error) {
    // Use server-provided retry-after if available
    if (error instanceof RateLimitError && error.retryAfter) {
      return error.retryAfter * 1000;
    }
    
    // Exponential backoff: 1s, 2s, 4s, 8s...
    const baseDelay = 1000;
    const exponentialDelay = baseDelay * Math.pow(2, attempt);
    
    // Add jitter to prevent thundering herd
    const jitter = Math.random() * 1000;
    
    return exponentialDelay + jitter;
  }

  sleep(ms) {
    return new Promise(resolve => setTimeout(resolve, ms));
  }
}
```

## Error Monitoring and Logging

### Server-Side Error Tracking
```http
# Include correlation IDs for tracking
POST /api/users
X-Request-ID: req_abc123

Response: 500 Internal Server Error
{
  "error": "internal_server_error",
  "message": "An unexpected error occurred",
  "request_id": "req_abc123",
  "error_id": "err_def456",
  "timestamp": "2025-08-27T10:30:00Z"
}

# Server logs (structured logging)
{
  "level": "error",
  "timestamp": "2025-08-27T10:30:00Z",
  "request_id": "req_abc123",
  "error_id": "err_def456",
  "user_id": "user_123",
  "endpoint": "POST /api/users",
  "error_type": "database_connection_failed",
  "stack_trace": "...",
  "user_agent": "MyApp/1.0",
  "ip_address": "192.168.1.100"
}
```

### Error Analytics
```http
# Track error patterns
{
  "error_metrics": {
    "total_errors_24h": 145,
    "error_rate": 0.025,
    "top_errors": [
      {
        "error_code": "validation_failed",
        "count": 89,
        "percentage": 61.4
      },
      {
        "error_code": "resource_not_found", 
        "count": 34,
        "percentage": 23.4
      },
      {
        "error_code": "rate_limit_exceeded",
        "count": 22,
        "percentage": 15.2
      }
    ],
    "endpoints_with_most_errors": [
      {
        "endpoint": "POST /api/users",
        "error_count": 56,
        "total_requests": 1200
      }
    ]
  }
}
```

## Interview Questions

1. **Q: What makes a good error response in REST APIs?**
   **A:** Good error responses include: appropriate HTTP status codes, consistent structure across the API, human-readable messages, machine-readable error codes, actionable details for fixing the error, request correlation IDs for tracking, and links to documentation.

2. **Q: How do you handle validation errors with multiple field issues?**
   **A:** Return 400 Bad Request with detailed field-level errors. Include field name, error code, message, and rejected value for each validation failure. Use consistent error structure and provide all validation errors at once rather than stopping at the first error.

3. **Q: What's the difference between 401 and 403 status codes?**
   **A:** 401 Unauthorized means authentication is missing or invalid (who you are problem). 403 Forbidden means authentication succeeded but authorization failed (what you can do problem). 401 requires login, 403 requires permission changes.

4. **Q: When should you use 422 Unprocessable Entity vs 400 Bad Request?**
   **A:** Use 400 for syntax/format errors (malformed JSON, wrong data types). Use 422 for semantic errors where request is well-formed but violates business rules (end date before start date, insufficient funds).

5. **Q: How do you implement proper error logging without exposing sensitive information?**
   **A:** Log detailed errors server-side with request IDs, stack traces, and context. Return sanitized errors to clients with generic messages for server errors. Use correlation IDs to link client responses with server logs. Never expose internal system details.

6. **Q: What strategies help prevent error information disclosure vulnerabilities?**
   **A:** Return generic messages for server errors, sanitize error details in production, use different error detail levels for development vs production, avoid exposing database errors or file paths, and implement proper error boundaries.

7. **Q: How should clients handle intermittent server errors (5xx)?**
   **A:** Implement exponential backoff with jitter, respect Retry-After headers, limit retry attempts, only retry idempotent operations, distinguish between retriable (500, 502, 503, 504) and non-retriable errors (501), and provide user feedback for ongoing issues.

8. **Q: What's the best practice for API versioning in error responses?**
   **A:** Include version information in error responses when relevant, maintain backward compatibility in error structure across versions, document error format changes between versions, and use content negotiation for version-specific error formats when needed.

9. **Q: How do you handle errors in bulk/batch operations?**
   **A:** Use 207 Multi-Status with individual status codes for each item, or return 200/400 with detailed success/failure information for each item. Include partial success indicators and allow clients to identify which specific items failed and why.

10. **Q: What error handling considerations apply to asynchronous operations?**
    **A:** Return 202 Accepted immediately, provide status URLs for checking progress, include error information in status responses, send webhooks for completion/failure notifications, implement timeouts for long-running operations, and provide ways to cancel or retry failed operations.
