# Caching & Performance

Caching is a critical performance optimization technique for REST APIs that reduces server load, improves response times, and enhances user experience. Understanding different caching strategies, HTTP caching headers, and cache invalidation patterns is essential for building high-performance APIs.

## Types of Caching

### 1. Client-Side Caching (Browser Cache)
The client (browser or application) stores responses locally.

```http
# Server sends cacheable response
GET /api/products/123

Response: 200 OK
Cache-Control: public, max-age=3600
ETag: "product-123-v5"
Last-Modified: Wed, 27 Aug 2025 10:00:00 GMT
Content-Type: application/json

{
  "id": 123,
  "name": "Wireless Headphones",
  "price": 99.99,
  "updated_at": "2025-08-27T10:00:00Z"
}

# Subsequent request within cache period
GET /api/products/123
If-None-Match: "product-123-v5"

Response: 304 Not Modified
Cache-Control: public, max-age=3600
ETag: "product-123-v5"
# No response body, client uses cached version
```

### 2. Proxy/CDN Caching
Intermediate proxies or CDNs cache responses for multiple clients.

```http
# CDN-friendly response
GET /api/public/countries

Response: 200 OK
Cache-Control: public, max-age=86400, immutable
ETag: "countries-v2"
Vary: Accept-Language
Content-Type: application/json

{
  "countries": [
    {"code": "US", "name": "United States"},
    {"code": "UK", "name": "United Kingdom"}
  ],
  "last_updated": "2025-08-27T00:00:00Z"
}

# CDN serves cached response to other users
# Cache headers indicate it can be cached publicly for 24 hours
```

### 3. Server-Side Caching
The server caches computed results or database queries.

```http
# Expensive computation result cached server-side
GET /api/analytics/sales-report?year=2025

Response: 200 OK
Cache-Control: private, max-age=1800
X-Cache: HIT
X-Cache-TTL: 1654
Content-Type: application/json

{
  "year": 2025,
  "total_sales": 1250000,
  "monthly_breakdown": [...],
  "computed_at": "2025-08-27T10:00:00Z",
  "computation_time_ms": 45
}
```

### 4. Database Query Caching
Caching database query results to reduce database load.

```http
# Frequently accessed user profile
GET /api/users/123/profile

Response: 200 OK
Cache-Control: private, max-age=300
X-DB-Cache: HIT
X-Query-Time: 2ms
Content-Type: application/json

{
  "user_id": 123,
  "profile": {
    "name": "John Doe",
    "avatar_url": "https://cdn.example.com/avatars/123.jpg",
    "bio": "Software Developer"
  },
  "cached_at": "2025-08-27T10:15:00Z"
}
```

## HTTP Caching Headers

### Cache-Control Directive
Controls how and for how long responses are cached.

```http
# Public caching (can be cached by anyone)
Cache-Control: public, max-age=3600

# Private caching (only client can cache)
Cache-Control: private, max-age=300

# No caching
Cache-Control: no-cache, no-store, must-revalidate

# Conditional caching
Cache-Control: public, max-age=0, must-revalidate

# Immutable content (never changes)
Cache-Control: public, max-age=31536000, immutable

# Stale-while-revalidate (serve stale content while updating)
Cache-Control: public, max-age=3600, stale-while-revalidate=86400
```

### Common Cache-Control Patterns

#### Static Assets
```http
# CSS, JS, images with versioned URLs
GET /api/assets/styles-v123.css

Response: 200 OK
Cache-Control: public, max-age=31536000, immutable
ETag: "styles-v123"
Content-Type: text/css
```

#### API Data with Different Sensitivity Levels
```http
# Public data
GET /api/public/blog-posts

Response: 200 OK
Cache-Control: public, max-age=1800
Vary: Accept, Accept-Language

# User-specific data
GET /api/users/me/dashboard
Authorization: Bearer token

Response: 200 OK
Cache-Control: private, max-age=300, must-revalidate

# Real-time data
GET /api/stock-prices/AAPL

Response: 200 OK
Cache-Control: no-cache, must-revalidate
# Always revalidate with server

# Sensitive data
GET /api/users/me/payment-methods
Authorization: Bearer token

Response: 200 OK
Cache-Control: no-cache, no-store, must-revalidate
Pragma: no-cache
Expires: 0
```

### ETag (Entity Tag)
Unique identifier for a specific version of a resource.

```http
# Server generates ETag based on content
GET /api/products/123

Response: 200 OK
ETag: "product-123-hash-abc123def456"
Cache-Control: public, max-age=3600
Content-Type: application/json

{
  "id": 123,
  "name": "Product Name",
  "version": 5,
  "updated_at": "2025-08-27T10:00:00Z"
}

# Client conditional request
GET /api/products/123
If-None-Match: "product-123-hash-abc123def456"

# If content hasn't changed
Response: 304 Not Modified
ETag: "product-123-hash-abc123def456"

# If content has changed
Response: 200 OK
ETag: "product-123-hash-xyz789ghi012"
Content-Type: application/json

{
  "id": 123,
  "name": "Updated Product Name",
  "version": 6,
  "updated_at": "2025-08-27T11:00:00Z"
}
```

### ETag Generation Strategies
```javascript
// Content-based ETag
const crypto = require('crypto');

function generateContentETag(data) {
  const hash = crypto.createHash('md5')
    .update(JSON.stringify(data))
    .digest('hex');
  return `"${hash}"`;
}

// Version-based ETag
function generateVersionETag(resourceId, version) {
  return `"${resourceId}-v${version}"`;
}

// Timestamp-based ETag
function generateTimestampETag(resourceId, updatedAt) {
  const timestamp = new Date(updatedAt).getTime();
  return `"${resourceId}-${timestamp}"`;
}

// Weak ETag (for semantically equivalent resources)
function generateWeakETag(resourceId, semanticVersion) {
  return `W/"${resourceId}-${semanticVersion}"`;
}
```

### Last-Modified Header
Indicates when the resource was last modified.

```http
# Server includes Last-Modified
GET /api/documents/456

Response: 200 OK
Last-Modified: Wed, 27 Aug 2025 10:00:00 GMT
Cache-Control: public, max-age=3600
Content-Type: application/json

{
  "id": 456,
  "title": "API Documentation",
  "content": "...",
  "updated_at": "2025-08-27T10:00:00Z"
}

# Client conditional request
GET /api/documents/456
If-Modified-Since: Wed, 27 Aug 2025 10:00:00 GMT

# If not modified since
Response: 304 Not Modified
Last-Modified: Wed, 27 Aug 2025 10:00:00 GMT

# If modified since
Response: 200 OK
Last-Modified: Wed, 27 Aug 2025 11:30:00 GMT
Content-Type: application/json

{
  "id": 456,
  "title": "Updated API Documentation",
  "content": "...",
  "updated_at": "2025-08-27T11:30:00Z"
}
```

## Caching Strategies

### 1. Time-Based Caching (TTL)
Cache expires after a fixed time period.

```http
# Short TTL for frequently changing data
GET /api/live-scores

Response: 200 OK
Cache-Control: public, max-age=30
X-Cache-Strategy: time-based
Content-Type: application/json

{
  "games": [
    {"id": 1, "home": "TeamA", "away": "TeamB", "score": "2-1"}
  ],
  "last_updated": "2025-08-27T10:30:00Z"
}

# Medium TTL for moderately changing data
GET /api/product-categories

Response: 200 OK
Cache-Control: public, max-age=1800
# Cached for 30 minutes

# Long TTL for rarely changing data
GET /api/system/timezones

Response: 200 OK
Cache-Control: public, max-age=86400
# Cached for 24 hours
```

### 2. Event-Based Cache Invalidation
Cache is invalidated when specific events occur.

```http
# Product data cached until update event
GET /api/products/123

Response: 200 OK
Cache-Control: public, max-age=3600
ETag: "product-123-v5"
X-Cache-Events: product.updated, product.deleted

# When product is updated, cache is invalidated
POST /api/products/123
Content-Type: application/json

{
  "name": "Updated Product Name",
  "price": 129.99
}

Response: 200 OK
ETag: "product-123-v6"
X-Cache-Invalidated: product-123
Content-Type: application/json

{
  "id": 123,
  "name": "Updated Product Name", 
  "price": 129.99,
  "version": 6
}
```

### 3. Hierarchical Caching
Related resources share cache dependencies.

```http
# User profile affects multiple cached resources
GET /api/users/123/profile

Response: 200 OK
Cache-Control: private, max-age=600
X-Cache-Tags: user:123, profile:123
Content-Type: application/json

{
  "user_id": 123,
  "name": "John Doe",
  "avatar_url": "...",
  "bio": "..."
}

# User's posts also tagged with user cache tag
GET /api/users/123/posts

Response: 200 OK
Cache-Control: private, max-age=300
X-Cache-Tags: user:123, posts:user:123
Content-Type: application/json

{
  "posts": [...],
  "user": {
    "id": 123,
    "name": "John Doe"
  }
}

# When user updates profile, all related caches invalidated
PUT /api/users/123/profile
# Invalidates: user:123 tag, affecting profile and posts
```

### 4. Conditional Requests
Use ETags and Last-Modified for efficient caching.

```http
# Client makes conditional request
GET /api/news/latest
If-None-Match: "news-latest-v15"
If-Modified-Since: Wed, 27 Aug 2025 09:00:00 GMT

# Content hasn't changed
Response: 304 Not Modified
ETag: "news-latest-v15"
Last-Modified: Wed, 27 Aug 2025 09:00:00 GMT
Cache-Control: public, max-age=900

# Content has changed
Response: 200 OK
ETag: "news-latest-v16"
Last-Modified: Wed, 27 Aug 2025 10:30:00 GMT
Cache-Control: public, max-age=900
Content-Type: application/json

{
  "articles": [...],
  "last_updated": "2025-08-27T10:30:00Z"
}
```

## Performance Optimization Techniques

### 1. Response Compression
Reduce bandwidth usage with compression.

```http
# Client indicates compression support
GET /api/large-dataset
Accept-Encoding: gzip, deflate, br

# Server responds with compressed data
Response: 200 OK
Content-Encoding: gzip
Content-Type: application/json
Content-Length: 1234
X-Uncompressed-Size: 5678
Cache-Control: public, max-age=3600

# Compressed JSON response (body is gzipped)
```

### 2. Partial Content (Range Requests)
Allow clients to request specific portions of large resources.

```http
# Client requests part of large file
GET /api/files/large-report.pdf
Range: bytes=0-1023

Response: 206 Partial Content
Content-Range: bytes 0-1023/102400
Content-Length: 1024
Accept-Ranges: bytes
Cache-Control: public, max-age=3600

# First 1KB of the file

# Client requests next chunk
GET /api/files/large-report.pdf
Range: bytes=1024-2047

Response: 206 Partial Content
Content-Range: bytes 1024-2047/102400
Content-Length: 1024
```

### 3. Field Selection
Allow clients to request only needed fields.

```http
# Request specific fields only
GET /api/users?fields=id,name,email

Response: 200 OK
Cache-Control: public, max-age=600
Vary: fields
Content-Type: application/json

{
  "users": [
    {"id": 1, "name": "John", "email": "john@example.com"},
    {"id": 2, "name": "Jane", "email": "jane@example.com"}
  ],
  "selected_fields": ["id", "name", "email"]
}

# Request with additional fields
GET /api/users?fields=id,name,email,created_at,last_login

Response: 200 OK
Vary: fields
Content-Type: application/json

{
  "users": [
    {
      "id": 1,
      "name": "John",
      "email": "john@example.com",
      "created_at": "2025-01-15T10:00:00Z",
      "last_login": "2025-08-27T09:30:00Z"
    }
  ]
}
```

### 4. Resource Embedding/Expansion
Reduce number of requests by embedding related data.

```http
# Basic request
GET /api/posts/123

Response: 200 OK
{
  "id": 123,
  "title": "My Blog Post",
  "author_id": 456,
  "category_id": 789,
  "created_at": "2025-08-27T10:00:00Z"
}

# Request with embedded resources
GET /api/posts/123?include=author,category,comments

Response: 200 OK
Cache-Control: private, max-age=300
Vary: include
Content-Type: application/json

{
  "id": 123,
  "title": "My Blog Post",
  "author_id": 456,
  "category_id": 789,
  "created_at": "2025-08-27T10:00:00Z",
  "author": {
    "id": 456,
    "name": "John Doe",
    "avatar_url": "..."
  },
  "category": {
    "id": 789,
    "name": "Technology",
    "slug": "technology"
  },
  "comments": [
    {
      "id": 1001,
      "content": "Great post!",
      "author": "Jane Smith"
    }
  ]
}
```

### 5. HTTP/2 Server Push
Proactively send resources the client will need.

```http
# Server pushes related resources
GET /api/dashboard

Response: 200 OK
# Server also pushes:
# - /api/users/me/notifications
# - /api/users/me/recent-activity
# - /api/system/status

Content-Type: application/json
{
  "user": {...},
  "quick_stats": {...},
  "pushed_resources": [
    "/api/users/me/notifications",
    "/api/users/me/recent-activity",
    "/api/system/status"
  ]
}
```

## Cache Invalidation Patterns

### 1. Tag-Based Invalidation
Invalidate multiple related cache entries using tags.

```javascript
// Cache with tags
const cacheEntry = {
  key: 'user:123:profile',
  data: userProfile,
  tags: ['user:123', 'profile', 'department:engineering'],
  expires: Date.now() + 3600000 // 1 hour
};

// Invalidate all caches tagged with 'user:123'
cache.invalidateByTag('user:123');

// Invalidate department-related caches
cache.invalidateByTag('department:engineering');
```

### 2. Write-Through Caching
Update cache immediately when data changes.

```http
# Update user profile
PUT /api/users/123/profile
Content-Type: application/json

{
  "name": "John Smith",
  "bio": "Updated bio"
}

Response: 200 OK
X-Cache-Updated: user:123:profile
Content-Type: application/json

{
  "id": 123,
  "name": "John Smith",
  "bio": "Updated bio",
  "updated_at": "2025-08-27T11:00:00Z"
}

# Subsequent request returns fresh cached data
GET /api/users/123/profile

Response: 200 OK
X-Cache: HIT-FRESH
ETag: "user-123-profile-v6"
```

### 3. Write-Behind Caching
Update cache immediately, database asynchronously.

```http
# Fast response from cache update
PUT /api/users/123/settings
Content-Type: application/json

{
  "theme": "dark",
  "notifications": false
}

Response: 200 OK
X-Cache-Status: UPDATED
X-DB-Status: QUEUED
Content-Type: application/json

{
  "theme": "dark",
  "notifications": false,
  "updated_at": "2025-08-27T11:00:00Z",
  "sync_status": "pending"
}
```

### 4. Cache-Aside Pattern
Application manages cache explicitly.

```javascript
// Cache-aside implementation
async function getUserProfile(userId) {
  const cacheKey = `user:${userId}:profile`;
  
  // Try cache first
  let profile = await cache.get(cacheKey);
  
  if (!profile) {
    // Cache miss - fetch from database
    profile = await database.getUserProfile(userId);
    
    // Store in cache for next time
    await cache.set(cacheKey, profile, 3600); // 1 hour TTL
  }
  
  return profile;
}

async function updateUserProfile(userId, updates) {
  // Update database
  const updatedProfile = await database.updateUserProfile(userId, updates);
  
  // Invalidate cache
  const cacheKey = `user:${userId}:profile`;
  await cache.delete(cacheKey);
  
  return updatedProfile;
}
```

## Advanced Caching Patterns

### 1. Multi-Level Caching
Different cache layers with different characteristics.

```http
# Request flow through multiple cache layers
GET /api/products/popular

# L1 Cache (In-Memory): 1ms lookup, 5-minute TTL
# L2 Cache (Redis): 5ms lookup, 1-hour TTL  
# L3 Cache (CDN): 50ms lookup, 6-hour TTL
# Database: 200ms lookup

Response: 200 OK
X-Cache-L1: MISS
X-Cache-L2: HIT
X-Cache-TTL-L1: 300
X-Cache-TTL-L2: 3300
Content-Type: application/json

{
  "products": [...],
  "cache_info": {
    "served_from": "L2_REDIS",
    "total_latency_ms": 5
  }
}
```

### 2. Cache Warming
Pre-populate cache with likely-to-be-requested data.

```http
# Cache warming endpoint
POST /api/cache/warm
Authorization: Bearer admin_token
Content-Type: application/json

{
  "strategies": [
    "popular_products",
    "recent_blog_posts", 
    "user_dashboards"
  ],
  "priority": "high"
}

Response: 202 Accepted
{
  "warming_job_id": "warm_abc123",
  "estimated_completion": "2025-08-27T11:05:00Z",
  "status_url": "/api/cache/warm/warm_abc123"
}

# Check warming status
GET /api/cache/warm/warm_abc123

Response: 200 OK
{
  "job_id": "warm_abc123",
  "status": "in_progress",
  "progress": {
    "completed": 1250,
    "total": 2000,
    "percentage": 62.5
  },
  "strategies": {
    "popular_products": "completed",
    "recent_blog_posts": "in_progress",
    "user_dashboards": "pending"
  }
}
```

### 3. Smart Cache Expiration
Adjust cache TTL based on data characteristics.

```javascript
function calculateSmartTTL(data, accessPattern) {
  const baseTTL = 3600; // 1 hour
  
  // Popular data gets longer TTL
  if (accessPattern.requestsPerHour > 100) {
    return baseeTTL * 4; // 4 hours
  }
  
  // Rarely accessed data gets shorter TTL
  if (accessPattern.requestsPerHour < 10) {
    return baseTTL / 2; // 30 minutes
  }
  
  // Time-sensitive data gets shorter TTL
  if (data.type === 'real_time') {
    return 60; // 1 minute
  }
  
  // Static data gets longer TTL
  if (data.type === 'static') {
    return baseTTL * 24; // 24 hours
  }
  
  return baseTTL;
}
```

### 4. Cache Stampede Prevention
Prevent multiple processes from regenerating the same cache entry.

```javascript
// Lock-based cache regeneration
async function getWithStampedeProtection(key, generator, ttl) {
  const lockKey = `lock:${key}`;
  const value = await cache.get(key);
  
  if (value) {
    return value;
  }
  
  // Try to acquire lock
  const lockAcquired = await cache.setNX(lockKey, 'locked', 30); // 30 sec lock
  
  if (lockAcquired) {
    try {
      // This process regenerates the cache
      const freshValue = await generator();
      await cache.set(key, freshValue, ttl);
      return freshValue;
    } finally {
      await cache.delete(lockKey);
    }
  } else {
    // Another process is regenerating, wait briefly and retry
    await sleep(100);
    return getWithStampedeProtection(key, generator, ttl);
  }
}
```

## Monitoring and Metrics

### Cache Performance Metrics
```http
# Cache statistics endpoint
GET /api/admin/cache/stats
Authorization: Bearer admin_token

Response: 200 OK
Content-Type: application/json

{
  "cache_stats": {
    "hit_rate": 0.847,
    "miss_rate": 0.153,
    "total_requests": 1000000,
    "hits": 847000,
    "misses": 153000,
    "evictions": 5420,
    "memory_usage": {
      "used_mb": 512,
      "total_mb": 1024,
      "percentage": 50.0
    },
    "average_response_time": {
      "cache_hit_ms": 2.5,
      "cache_miss_ms": 45.2,
      "overall_ms": 8.7
    }
  },
  "by_resource_type": {
    "users": {"hit_rate": 0.92, "requests": 250000},
    "products": {"hit_rate": 0.78, "requests": 300000},
    "orders": {"hit_rate": 0.65, "requests": 150000}
  },
  "time_window": "24h"
}
```

### Cache Headers for Debugging
```http
# Development headers for cache debugging
GET /api/products/123
X-Debug-Cache: true

Response: 200 OK
X-Cache: HIT
X-Cache-Age: 1847
X-Cache-TTL: 1753
X-Cache-Key: product:123:v5
X-Cache-Tags: product,category:electronics
X-Cache-Size: 2048
X-Generation-Time: 0ms
X-DB-Query-Time: N/A
ETag: "product-123-v5"
Cache-Control: public, max-age=3600
```

## Interview Questions

1. **Q: What's the difference between ETags and Last-Modified headers?**
   **A:** ETags are content-based identifiers that change when resource content changes, supporting both strong and weak validation. Last-Modified uses timestamps, which have 1-second granularity and may not catch rapid changes. ETags are more reliable for cache validation.

2. **Q: When should you use private vs public cache-control directives?**
   **A:** Use 'private' for user-specific data that shouldn't be cached by shared proxies/CDNs (like personal profiles). Use 'public' for data that can be cached by anyone (like product listings, public content). Private caches respect user context.

3. **Q: How do you handle cache invalidation in distributed systems?**
   **A:** Use cache tags for related resource invalidation, implement event-driven cache clearing, use distributed cache invalidation patterns (pub/sub), consider eventual consistency trade-offs, and implement cache versioning for gradual rollouts.

4. **Q: What are the performance implications of different caching strategies?**
   **A:** Time-based caching is simple but may serve stale data. Event-based invalidation is accurate but complex. Write-through ensures consistency but slower writes. Write-behind is fast but risks data loss. Choose based on consistency vs performance requirements.

5. **Q: How do you prevent cache stampede problems?**
   **A:** Use lock-based regeneration, implement stale-while-revalidate patterns, use background cache warming, implement jittered expiration times, and consider queue-based cache regeneration for high-traffic scenarios.

6. **Q: What caching considerations apply to paginated responses?**
   **A:** Cache each page separately with appropriate Vary headers, consider total count caching separately, implement smart invalidation for data changes, handle page size variations, and consider cursor-based pagination for better cache efficiency.

7. **Q: How do you implement cache warming effectively?**
   **A:** Identify frequently accessed data patterns, implement predictive pre-loading, use background jobs for warming, prioritize based on access patterns, monitor warming effectiveness, and avoid overwhelming systems during warming.

8. **Q: What are the security implications of caching?**
   **A:** Ensure private data isn't cached publicly, implement proper cache key isolation, consider information leakage through cache timing, secure cache infrastructure, and implement cache poisoning prevention.

9. **Q: How do you handle cache coherence across multiple API instances?**
   **A:** Use shared cache stores (Redis), implement distributed cache invalidation, use event streaming for consistency, consider cache partitioning strategies, and implement proper cache synchronization protocols.

10. **Q: What metrics should you monitor for cache performance?**
    **A:** Monitor hit/miss ratios, cache response times, memory usage, eviction rates, cache size distribution, invalidation patterns, and business impact metrics like overall API response times and database load reduction.
