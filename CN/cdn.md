# CDN

Brief: Content Delivery Networks cache content closer to users to reduce latency and offload origin.

## Key concepts
- Edge servers/POPs, cache keys, TTL, invalidation, cache-control headers
- Purge, stale-while-revalidate, origin shield
- Anycast routing to nearest POP

## Practical example
- Serve static assets via CDN with long TTL and versioned URLs.

## Common interview Q&A
- How CDN differs from load balancer? How handle cache busting?
