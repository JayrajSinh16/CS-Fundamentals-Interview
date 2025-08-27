# HTTP Request/Response Structure

Understand the on-the-wire structure before diving into payload specifics.

## Request
- Start line: METHOD SP REQUEST-TARGET SP HTTP-VERSION (e.g., `GET /orders/123 HTTP/1.1`)
- Headers: key-value pairs describing metadata (Host, Accept, Authorization, etc.)
- Empty line
- Body: optional (POST/PUT/PATCH usually include JSON/XML/form-data)

Example:
```http
POST /orders HTTP/1.1
Host: api.example.com
Content-Type: application/json
Accept: application/json
Authorization: Bearer <token>

{ "items": [{"sku":"A1","qty":2}] }
```

## Response
- Status line: HTTP-VERSION SP STATUS-CODE SP REASON-PHRASE (e.g., `HTTP/1.1 201 Created`)
- Headers: metadata (Content-Type, Cache-Control, ETag, Location)
- Empty line
- Body: optional (JSON for success/error per your API guidelines)

Example:
```http
HTTP/1.1 201 Created
Content-Type: application/json
Location: /orders/123
ETag: "v1-abc123"

{ "id": 123, "status": "pending" }
```

For body design patterns and conventions, see: [Request/Response Body Structure](./request-response-body.md).
