# Filtering & Sorting

APIs should provide flexible filtering and sorting while staying secure and predictable.

## Filtering Patterns

- Simple equality: ?status=active
- Multiple values: ?status=active,pending or repeated params
- Ranges: ?price_min=10&price_max=100
- Dates: ?from=2025-01-01&to=2025-01-31 (use ISO 8601)
- Nested fields: ?customer.id=123

## Sorting

- Single field: ?sort=created_at (asc default)
- Descending: ?sort=-created_at
- Multiple fields: ?sort=created_at,-price

## Examples

```http
GET /api/products?category=books&price_min=10&price_max=50&sort=-rating,price&in_stock=true&limit=20&page=2

200 OK
{
  "data": [ /* products */ ],
  "filters": { "category": "books", "price_min": 10, "price_max": 50, "in_stock": true },
  "sort": ["-rating", "price"],
  "pagination": { "page": 2, "per_page": 20, "total": 523 }
}
```

## Security

- Whitelist filterable and sortable fields
- Validate operators and types
- Prevent SQL injection by using parameterized queries
- Enforce limits to avoid expensive scans

## Advanced

- RSQL/FiQL-like syntax when necessary (document well)
- Sparse fieldsets: ?fields= id,name,price
- Relationship filters: ?seller_id=... or via nested paths

## Interview Questions

1. How to design a consistent filtering API?
2. How to prevent abuse of filtering/sorting queries?
3. Pagination + sorting consistency—what can go wrong?
4. How to implement sparse fieldsets and projections?
5. How would you expose complex search vs simple filters?
