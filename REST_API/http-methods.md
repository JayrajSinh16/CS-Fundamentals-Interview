# HTTP Methods & CRUD Mapping

HTTP methods define the action to be performed on a resource. Each method has specific semantics and should be used consistently to create predictable, RESTful APIs. Understanding proper CRUD mapping and method properties is essential for API design interviews.

## HTTP Methods Overview

### Safe and Idempotent Properties
- **Safe**: Method doesn't modify server state (no side effects)
- **Idempotent**: Multiple identical requests have the same effect as a single request

| Method | Safe | Idempotent | Cacheable | Request Body | Response Body |
|--------|------|------------|-----------|--------------|---------------|
| GET | ✅ | ✅ | ✅ | Optional | ✅ |
| POST | ❌ | ❌ | ❌ | ✅ | ✅ |
| PUT | ❌ | ✅ | ❌ | ✅ | ✅ |
| PATCH | ❌ | ❌ | ❌ | ✅ | ✅ |
| DELETE | ❌ | ✅ | ❌ | Optional | Optional |
| HEAD | ✅ | ✅ | ✅ | ❌ | ❌ |
| OPTIONS | ✅ | ✅ | ❌ | Optional | ✅ |

## Detailed Method Descriptions

### GET - Retrieve Resource
**Purpose**: Retrieve representation of a resource
**CRUD**: Read
**Safe**: Yes | **Idempotent**: Yes

```http
# Get single resource
GET /api/users/123
Accept: application/json

Response: 200 OK
{
  "id": 123,
  "name": "John Doe",
  "email": "john@example.com",
  "created_at": "2025-08-27T10:00:00Z"
}

# Get collection with filtering
GET /api/users?status=active&role=developer&page=1&limit=10

Response: 200 OK
{
  "data": [
    {"id": 123, "name": "John Doe"},
    {"id": 124, "name": "Jane Smith"}
  ],
  "pagination": {
    "page": 1,
    "limit": 10,
    "total": 25,
    "pages": 3
  },
  "filters": {
    "status": "active",
    "role": "developer"
  }
}
```

### POST - Create Resource
**Purpose**: Create new resource or perform action
**CRUD**: Create
**Safe**: No | **Idempotent**: No

```http
# Create new user
POST /api/users
Content-Type: application/json

{
  "name": "John Doe",
  "email": "john@example.com",
  "role": "developer"
}

Response: 201 Created
Location: /api/users/123
{
  "id": 123,
  "name": "John Doe",
  "email": "john@example.com",
  "role": "developer",
  "created_at": "2025-08-27T10:30:00Z"
}

# Create nested resource
POST /api/users/123/posts
Content-Type: application/json

{
  "title": "My First Blog Post",
  "content": "This is the content of my post.",
  "status": "draft"
}

Response: 201 Created
Location: /api/users/123/posts/456
```

### PUT - Update/Replace Resource
**Purpose**: Update existing resource or create if doesn't exist
**CRUD**: Update (full replacement)
**Safe**: No | **Idempotent**: Yes

```http
# Update existing user (full replacement)
PUT /api/users/123
Content-Type: application/json

{
  "name": "John Smith",
  "email": "john.smith@example.com",
  "role": "senior_developer",
  "bio": "Experienced developer"
}

Response: 200 OK
{
  "id": 123,
  "name": "John Smith",
  "email": "john.smith@example.com",
  "role": "senior_developer",
  "bio": "Experienced developer",
  "updated_at": "2025-08-27T11:00:00Z"
}

# Create resource if doesn't exist
PUT /api/users/999
Content-Type: application/json

{
  "name": "New User",
  "email": "new@example.com"
}

Response: 201 Created
Location: /api/users/999
```

### PATCH - Partial Update
**Purpose**: Apply partial modifications to resource
**CRUD**: Update (partial)
**Safe**: No | **Idempotent**: Depends on implementation

```http
# Update specific fields only
PATCH /api/users/123
Content-Type: application/json

{
  "email": "newemail@example.com",
  "role": "lead_developer"
}

Response: 200 OK
{
  "id": 123,
  "name": "John Smith",  # Unchanged
  "email": "newemail@example.com",  # Updated
  "role": "lead_developer",  # Updated
  "bio": "Experienced developer",  # Unchanged
  "updated_at": "2025-08-27T11:15:00Z"
}

# JSON Patch format (RFC 6902)
PATCH /api/users/123
Content-Type: application/json-patch+json

[
  { "op": "replace", "path": "/email", "value": "new@example.com" },
  { "op": "add", "path": "/phone", "value": "+1234567890" },
  { "op": "remove", "path": "/bio" }
]
```

### DELETE - Remove Resource
**Purpose**: Remove resource
**CRUD**: Delete
**Safe**: No | **Idempotent**: Yes

```http
# Delete specific resource
DELETE /api/users/123

Response: 204 No Content

# Delete with confirmation
DELETE /api/users/123?confirm=true

Response: 200 OK
{
  "message": "User deleted successfully",
  "deleted_at": "2025-08-27T11:30:00Z"
}

# Soft delete
DELETE /api/users/123

Response: 200 OK
{
  "id": 123,
  "status": "deleted",
  "deleted_at": "2025-08-27T11:30:00Z"
}
```

### HEAD - Metadata Only
**Purpose**: Get response headers without body
**Safe**: Yes | **Idempotent**: Yes

```http
# Check if resource exists and get metadata
HEAD /api/users/123

Response: 200 OK
Content-Type: application/json
Content-Length: 245
Last-Modified: Wed, 27 Aug 2025 11:00:00 GMT
ETag: "abc123"
```

### OPTIONS - Discover Capabilities
**Purpose**: Get allowed methods and capabilities
**Safe**: Yes | **Idempotent**: Yes

```http
# Discover allowed methods
OPTIONS /api/users/123

Response: 200 OK
Allow: GET, PUT, PATCH, DELETE, HEAD, OPTIONS
Access-Control-Allow-Methods: GET, PUT, PATCH, DELETE
Access-Control-Allow-Headers: Content-Type, Authorization
```

## CRUD to HTTP Method Mapping

### Complete User CRUD Example

```http
# CREATE - Create new user
POST /api/users
{
  "name": "John Doe",
  "email": "john@example.com"
}
→ 201 Created

# READ - Get single user
GET /api/users/123
→ 200 OK

# READ - Get all users
GET /api/users
→ 200 OK

# UPDATE - Full replacement
PUT /api/users/123
{
  "name": "John Smith",
  "email": "john.smith@example.com"
}
→ 200 OK

# UPDATE - Partial update
PATCH /api/users/123
{
  "email": "newemail@example.com"
}
→ 200 OK

# DELETE - Remove user
DELETE /api/users/123
→ 204 No Content
```

## Advanced Method Usage

### Bulk Operations
```http
# Create multiple resources
POST /api/users/bulk
Content-Type: application/json

{
  "users": [
    {"name": "User 1", "email": "user1@example.com"},
    {"name": "User 2", "email": "user2@example.com"}
  ]
}

Response: 201 Created
{
  "created": [
    {"id": 123, "name": "User 1"},
    {"id": 124, "name": "User 2"}
  ]
}

# Update multiple resources
PATCH /api/users/bulk
Content-Type: application/json

{
  "updates": [
    {"id": 123, "role": "admin"},
    {"id": 124, "status": "inactive"}
  ]
}

Response: 200 OK
```

### Action Resources
```http
# Non-CRUD actions using POST
POST /api/users/123/activate
→ 200 OK

POST /api/users/123/send-notification
Content-Type: application/json
{
  "type": "welcome",
  "message": "Welcome to our platform!"
}
→ 200 OK

# Password reset
POST /api/users/reset-password
{
  "email": "john@example.com"
}
→ 202 Accepted
```

### Conditional Requests
```http
# Update only if not modified (using ETag)
PUT /api/users/123
If-Match: "abc123"
Content-Type: application/json

{
  "name": "Updated Name"
}

# If ETag doesn't match
Response: 412 Precondition Failed

# Delete only if unmodified since date
DELETE /api/users/123
If-Unmodified-Since: Wed, 27 Aug 2025 10:00:00 GMT
```

## Common Mistakes to Avoid

### 1. Using Wrong Methods for Actions
```http
# Wrong
GET /api/users/123/delete
POST /api/users/getAll

# Correct
DELETE /api/users/123
GET /api/users
```

### 2. Not Using Proper Status Codes
```http
# Wrong
DELETE /api/users/123
Response: 200 OK {"message": "deleted"}

# Correct
DELETE /api/users/123
Response: 204 No Content
```

### 3. Misusing PUT vs PATCH
```http
# Wrong - Using PATCH for full replacement
PATCH /api/users/123
{
  "name": "John",
  "email": "john@example.com",
  "role": "developer",
  "bio": "Full bio here"
}

# Correct - Use PUT for full replacement
PUT /api/users/123
{
  "name": "John",
  "email": "john@example.com",
  "role": "developer",
  "bio": "Full bio here"
}

# Correct - Use PATCH for partial updates
PATCH /api/users/123
{
  "role": "senior_developer"
}
```

### 4. Breaking Idempotency
```http
# Wrong - PUT should be idempotent
PUT /api/users/123
{
  "login_count": 15  # Incrementing counter breaks idempotency
}

# Correct - Use POST for non-idempotent operations
POST /api/users/123/increment-login-count
```

## cURL Examples

```bash
# GET request
curl -X GET "https://api.example.com/users/123" \
  -H "Accept: application/json" \
  -H "Authorization: Bearer token123"

# POST request
curl -X POST "https://api.example.com/users" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer token123" \
  -d '{
    "name": "John Doe",
    "email": "john@example.com"
  }'

# PUT request
curl -X PUT "https://api.example.com/users/123" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer token123" \
  -d '{
    "name": "John Smith",
    "email": "john.smith@example.com"
  }'

# PATCH request
curl -X PATCH "https://api.example.com/users/123" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer token123" \
  -d '{
    "email": "newemail@example.com"
  }'

# DELETE request
curl -X DELETE "https://api.example.com/users/123" \
  -H "Authorization: Bearer token123"

# HEAD request
curl -I "https://api.example.com/users/123" \
  -H "Authorization: Bearer token123"

# OPTIONS request
curl -X OPTIONS "https://api.example.com/users/123" \
  -H "Authorization: Bearer token123"
```

## Interview Questions

1. **Q: Explain the difference between PUT and PATCH methods.**
   **A:** PUT replaces the entire resource (full update) and is idempotent. PATCH applies partial modifications and may not be idempotent. PUT requires sending all fields, PATCH only sends fields to update.

2. **Q: Why is GET considered safe and idempotent?**
   **A:** Safe because it doesn't modify server state (no side effects). Idempotent because multiple identical GET requests return the same result without changing the resource.

3. **Q: When would you use POST vs PUT for creating resources?**
   **A:** Use POST when the server assigns the resource ID (e.g., POST /api/users). Use PUT when the client specifies the ID (e.g., PUT /api/users/123). POST is not idempotent; PUT is idempotent.

4. **Q: What's the purpose of the OPTIONS method?**
   **A:** Discovers allowed methods and capabilities for a resource. Commonly used in CORS preflight requests to check if a cross-origin request is allowed before sending the actual request.

5. **Q: How do you handle bulk operations in REST?**
   **A:** Use POST to a collection endpoint with an array of resources (POST /api/users/bulk) or create a specific bulk resource endpoint. Return appropriate status codes and indicate partial success/failure.

6. **Q: What makes DELETE idempotent even though it modifies state?**
   **A:** The result is the same regardless of how many times you call it - the resource is gone. The first DELETE removes it, subsequent DELETEs have no effect (resource already doesn't exist).

7. **Q: How should you handle non-CRUD operations in REST?**
   **A:** Model them as resources or use POST for actions. Examples: POST /api/users/123/activate, POST /api/orders/123/cancel. Avoid verbs in URLs when possible.

8. **Q: What's the difference between 200 and 204 status codes for successful operations?**
   **A:** 200 OK includes a response body with data. 204 No Content indicates success but no response body (common for DELETE operations or successful updates with no data to return).

9. **Q: How do conditional requests work with HTTP methods?**
   **A:** Use headers like If-Match (ETag), If-None-Match, If-Modified-Since to make requests conditional. Prevents race conditions and unnecessary transfers. Returns 304 Not Modified or 412 Precondition Failed.

10. **Q: Why is POST not idempotent?**
    **A:** Each POST request typically creates a new resource or triggers a new action, resulting in different outcomes. Multiple POST requests to create users would create multiple users with different IDs.
