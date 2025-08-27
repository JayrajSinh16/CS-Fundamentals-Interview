# Search & Complex Queries

Provide search capabilities beyond simple filtering when needed.

## Options

- Simple search: q=term across key fields
- Advanced filters: multiple fields, ranges, boolean ops
- Full-text search: integrate with Elasticsearch/OpenSearch
- Facets and aggregations for analytics

## Examples

```http
# Simple
GET /api/products/search?q=laptop

# Advanced
GET /api/products/search?brand=apple&price_min=1000&price_max=2000&sort=-rating

# Full-text with facets (backend powered)
GET /api/products/search?q=ultrabook&facets=brand,price_range

200 OK
{
  "results": [ /* products */ ],
  "facets": {
    "brand": [{ "value": "Apple", "count": 42 }, { "value": "Dell", "count": 31 }],
    "price_range": [{ "value": "1000-1500", "count": 50 }]
  },
  "total": 1234
}
```

## Considerations

- Relevance scoring and tie-breaking
- Pagination and deep paging issues
- Caching search results (keyed by query)
- Security: field allow-lists, query parsing

## Interview Questions

1. When to add a dedicated search endpoint vs filters?
2. How to design facets/aggregations responses?
3. How to avoid deep paging performance issues?
4. How to secure complex query parsing?
5. How to measure and tune search relevance?
