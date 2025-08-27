# Caching Strategies

Caching improves performance and reduces load. Use HTTP caching headers and appropriate cache layers.

## HTTP Caching Headers

- Cache-Control: max-age, s-maxage, no-store, no-cache, private, public
- ETag / If-None-Match: revalidation via validators
- Last-Modified / If-Modified-Since: time-based validators
- Vary: content negotiation (Accept, Accept-Language, Authorization)

## Examples

```http
GET /api/users/123
If-None-Match: "W/\"etag-abc\""

304 Not Modified
ETag: "W/\"etag-abc\""
Cache-Control: max-age=600, public

# CDN-specific
Cache-Control: s-maxage=3600, stale-while-revalidate=60, stale-if-error=300
```

## Layers

- Client cache (browser)
- CDN/edge cache
- Reverse proxy (Varnish, NGINX)
- Application cache (Redis, in-memory)

## Invalidation

- Time-based expiry
- ETag revalidation
- Cache busting via versioned URLs
- Purge on write: purge related keys in app cache or CDN

## Interview Questions

1. ETag vs Last-Modified differences?
2. How to design cache-friendly URLs?
3. How to cache authenticated responses safely?
4. What’s stale-while-revalidate and stale-if-error?
5. When to choose write-through vs cache-aside?
