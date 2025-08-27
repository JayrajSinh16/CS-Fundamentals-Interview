# Path vs Query Parameters

Choosing between path and query parameters affects URL semantics, cacheability, and clarity. Use them consistently to convey meaning.

## Quick Rules

- Path parameters: Identify a specific resource or hierarchical segment
- Query parameters: Filter, sort, paginate, or modify the representation

## Examples

```http
# Path parameters (resource identity)
GET /users/{userId}
GET /users/{userId}/orders/{orderId}
GET /stores/{storeId}/products

# Query parameters (representation controls)
GET /users?role=admin&active=true
GET /products?category=books&sort=price&order=asc&limit=20&page=2
GET /orders?from=2025-01-01&to=2025-01-31&status=shipped
```

## Edge Cases

- Search endpoints can be top-level: GET /search?q=laptop
- Complex filters: prefer query params or POST /search with body when filters are complex
- Composite IDs: either use path segments or a canonical encoded id

## Cacheability

- Both can be cached, but CDNs often cache path-based URLs by default
- Ensure query param whitelisting in CDN to prevent cache fragmentation

## Validation

- Path params: validate presence, type, format (UUID, numeric)
- Query params: validate ranges, enumerations, and defaults

## Interview Questions

1. When to use path vs query parameters?
2. How do query params affect caching?
3. How to represent actions in REST paths?
4. How do you design complex filtering?
5. How to validate and document parameters in OpenAPI?
