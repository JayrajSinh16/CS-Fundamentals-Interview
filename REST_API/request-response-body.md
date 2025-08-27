# Request/Response Body Structure

Designing clear and consistent request/response bodies helps clients integrate quickly and reduces ambiguity.

## Request Body Design

- Use JSON objects at the top level (not arrays)
- Validate types, required fields, and formats
- Avoid nested depth > 3 if possible; split into sub-resources
- Example (create user):
```http
POST /api/users
Content-Type: application/json

{
  "name": "Jane",
  "email": "jane@example.com",
  "roles": ["user"],
  "profile": {
    "bio": "Hello",
    "timezone": "UTC"
  }
}
```

## Response Body Design

- Include resource representation plus metadata when useful
- Use consistent casing (snake_case or camelCase) across API
- Include timestamps in ISO 8601
- Example:
```http
201 Created
Location: /api/users/124

{
  "id": 124,
  "name": "Jane",
  "email": "jane@example.com",
  "roles": ["user"],
  "profile": {
    "bio": "Hello",
    "timezone": "UTC"
  },
  "created_at": "2025-08-27T10:30:00Z",
  "links": {
    "self": "/api/users/124",
    "posts": "/api/users/124/posts"
  }
}
```

## Pagination Shape

```json
{
  "data": [ /* items */ ],
  "pagination": {
    "page": 2,
    "per_page": 20,
    "total": 523,
    "total_pages": 27,
    "next": "/api/users?page=3&per_page=20",
    "prev": "/api/users?page=1&per_page=20"
  }
}
```

## Error Shape

```json
{
  "error": "validation_failed",
  "message": "Request validation failed",
  "details": [ { "field": "email", "code": "invalid_format" } ],
  "request_id": "req_abc123",
  "timestamp": "2025-08-27T10:30:00Z"
}
```

## Partial Updates

- Use PATCH with sparse field updates
- Support JSON Merge Patch or JSON Patch when needed

```http
PATCH /api/users/124
Content-Type: application/merge-patch+json

{
  "profile": { "bio": "Updated" }
}
```

## Interview Questions

1. Why prefer object top-level over array in JSON?
2. Show a consistent pagination response shape.
3. Compare JSON Patch vs Merge Patch.
4. How to design error envelopes for machine/human readability?
5. How to embed links (HATEOAS) in responses pragmatically?
