# Complete CRUD Operations

Standard CRUD patterns for REST APIs with examples and best practices.

## Users Resource

```http
# Create
POST /api/users
201 Created
Location: /api/users/123
{ "id": 123, "name": "Jane", "email": "jane@example.com" }

# Read (get one)
GET /api/users/123
200 OK
{ "id": 123, "name": "Jane", "email": "jane@example.com" }

# Read (list)
GET /api/users?limit=20&page=1
200 OK
{ "data": [ ... ], "pagination": { ... } }

# Update (replace)
PUT /api/users/123
200 OK
{ "id": 123, "name": "Jane Doe", "email": "jane@example.com" }

# Partial Update
PATCH /api/users/123
200 OK
{ "id": 123, "name": "Jane Doe" }

# Delete
DELETE /api/users/123
204 No Content
```

## Best Practices

- Use proper status codes and Location on create
- Validate inputs; return consistent shapes
- Support filtering/sorting/pagination on list
- ETags for concurrency control
- Idempotency for PUT/DELETE

## Interview Questions

1. PUT vs PATCH differences and when to use each?
2. How to handle partial updates safely?
3. How to design bulk CRUD endpoints?
4. How to prevent lost updates in concurrent edits?
5. What headers improve CRUD ergonomics?
