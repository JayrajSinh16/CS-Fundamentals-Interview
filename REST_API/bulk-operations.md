# Bulk Operations

Bulk endpoints improve efficiency by handling multiple items in one request.

## Patterns

- Bulk create: POST /items/bulk
- Bulk update: PATCH /items/bulk
- Bulk delete: DELETE /items/bulk (body with ids or via query)

## Example

```http
POST /api/users/bulk
Content-Type: application/json

{
  "users": [
    { "name": "A", "email": "a@example.com" },
    { "name": "B", "email": "b@example.com" },
    { "name": "", "email": "invalid" }
  ]
}

207 Multi-Status
{
  "summary": { "total": 3, "successful": 2, "failed": 1 },
  "results": [
    { "index": 0, "status": 201, "id": 1 },
    { "index": 1, "status": 201, "id": 2 },
    { "index": 2, "status": 400, "error": { "field": "name", "code": "required" } }
  ]
}
```

## Considerations

- Transactional semantics (all-or-nothing vs per-item)
- Partial success reporting (207)
- Limits per request
- Idempotency keys per item

## Interview Questions

1. How to design idempotent bulk operations?
2. When to choose 200 vs 207 for bulk results?
3. How to handle validation and transactions for bulk?
4. How to paginate bulk input? Batching strategies?
5. How to stream large bulk results?
