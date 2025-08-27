# Rate Limiting

Rate limiting protects your API from abuse and ensures fair usage across clients.

## Headers

- X-RateLimit-Limit: total allowed in window
- X-RateLimit-Remaining: remaining requests
- X-RateLimit-Reset: UNIX timestamp when window resets
- Retry-After: seconds to wait (for 429 responses)

## Algorithms

- Fixed window
- Sliding window
- Token bucket (leaky bucket)

## Example

```http
GET /api/data
X-API-Key: key_123

200 OK
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 998
X-RateLimit-Reset: 1693126800

# Exceeded
GET /api/data

429 Too Many Requests
Retry-After: 60
{
  "error": "rate_limit_exceeded",
  "message": "Too many requests"
}
```

## Dimensions

- Per API key/user/IP
- Per endpoint (e.g., login stricter)
- Burst vs sustained

## Distributed Systems

- Use Redis for counters
- Consistent hashing for sharding
- Sync limits across instances

## Interview Questions

1. Token bucket vs sliding window trade-offs?
2. How to handle rate limits per endpoint and per user?
3. How to expose rate limit info to clients?
4. How to protect login endpoints specifically?
5. How to scale limits across multiple servers?
