# Pagination Strategies

Pagination is essential for handling large datasets in REST APIs. It allows clients to retrieve data in manageable chunks, improves performance, reduces memory usage, and provides better user experience. Understanding different pagination strategies and their trade-offs is crucial for building scalable APIs.

## Why Pagination is Important

### Performance Benefits
- **Reduced Response Size**: Smaller payloads transfer faster
- **Lower Memory Usage**: Server and client use less memory
- **Faster Query Execution**: Database queries on smaller result sets
- **Better Caching**: Smaller responses cache more effectively

### User Experience
- **Faster Initial Load**: First page loads quickly
- **Progressive Loading**: Users can start consuming data immediately
- **Predictable Performance**: Consistent response times regardless of total data size

### Resource Management
- **Rate Limiting**: Easier to implement fair usage policies
- **Server Load**: Distributes load across multiple requests
- **Database Impact**: Reduces database load and connection time

## Pagination Strategies

### 1. Offset-Based Pagination (Page Number)

Most common and intuitive pagination method using page numbers.

```http
# Get first page (page 1)
GET /api/users?page=1&limit=20

Response: 200 OK
Content-Type: application/json
{
  "data": [
    {"id": 1, "name": "John Doe", "email": "john@example.com"},
    {"id": 2, "name": "Jane Smith", "email": "jane@example.com"}
    // ... 18 more users
  ],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 1500,
    "pages": 75,
    "has_next": true,
    "has_prev": false,
    "next_page": 2,
    "prev_page": null
  },
  "links": {
    "self": "/api/users?page=1&limit=20",
    "first": "/api/users?page=1&limit=20",
    "last": "/api/users?page=75&limit=20",
    "next": "/api/users?page=2&limit=20",
    "prev": null
  }
}

# Get specific page
GET /api/users?page=5&limit=20

Response: 200 OK
{
  "data": [
    {"id": 81, "name": "User 81", "email": "user81@example.com"}
    // ... 19 more users
  ],
  "pagination": {
    "page": 5,
    "limit": 20, 
    "total": 1500,
    "pages": 75,
    "has_next": true,
    "has_prev": true,
    "next_page": 6,
    "prev_page": 4
  },
  "links": {
    "self": "/api/users?page=5&limit=20",
    "first": "/api/users?page=1&limit=20",
    "last": "/api/users?page=75&limit=20", 
    "next": "/api/users?page=6&limit=20",
    "prev": "/api/users?page=4&limit=20"
  }
}
```

**Advantages:**
- Intuitive and user-friendly
- Easy to implement
- Supports jumping to any page
- Works well with UI pagination controls

**Disadvantages:**
- Performance degrades with large offsets
- Inconsistent results if data changes during pagination
- Expensive COUNT queries for total count

### 2. Offset-Based Pagination (Limit/Offset)

Similar to page-based but uses explicit offset values.

```http
# Get first 20 records
GET /api/products?limit=20&offset=0

Response: 200 OK
{
  "data": [
    {"id": 1, "name": "Product 1", "price": 29.99},
    {"id": 2, "name": "Product 2", "price": 39.99"}
    // ... 18 more products
  ],
  "pagination": {
    "limit": 20,
    "offset": 0,
    "total": 5000,
    "has_more": true
  },
  "links": {
    "self": "/api/products?limit=20&offset=0",
    "next": "/api/products?limit=20&offset=20"
  }
}

# Get next 20 records
GET /api/products?limit=20&offset=20

Response: 200 OK
{
  "data": [
    {"id": 21, "name": "Product 21", "price": 49.99"}
    // ... 19 more products
  ],
  "pagination": {
    "limit": 20,
    "offset": 20,
    "total": 5000,
    "has_more": true
  },
  "links": {
    "self": "/api/products?limit=20&offset=20",
    "prev": "/api/products?limit=20&offset=0",
    "next": "/api/products?limit=20&offset=40"
  }
}
```

### 3. Cursor-Based Pagination

Uses opaque cursors to navigate through data, more efficient for large datasets.

```http
# Initial request (no cursor)
GET /api/posts?limit=20

Response: 200 OK
{
  "data": [
    {
      "id": 1001,
      "title": "First Post",
      "created_at": "2025-08-27T10:00:00Z",
      "cursor": "eyJpZCI6MTAwMSwiY3JlYXRlZF9hdCI6IjIwMjUtMDgtMjdUMTA6MDA6MDBaIn0="
    },
    {
      "id": 1002,
      "title": "Second Post", 
      "created_at": "2025-08-27T10:05:00Z",
      "cursor": "eyJpZCI6MTAwMiwiY3JlYXRlZF9hdCI6IjIwMjUtMDgtMjdUMTA6MDU6MDBaIn0="
    }
    // ... 18 more posts
  ],
  "pagination": {
    "limit": 20,
    "has_next": true,
    "has_prev": false,
    "next_cursor": "eyJpZCI6MTAyMCwiY3JlYXRlZF9hdCI6IjIwMjUtMDgtMjdUMTE6MDA6MDBaIn0=",
    "prev_cursor": null
  },
  "links": {
    "self": "/api/posts?limit=20",
    "next": "/api/posts?limit=20&cursor=eyJpZCI6MTAyMCwiY3JlYXRlZF9hdCI6IjIwMjUtMDgtMjdUMTE6MDA6MDBaIn0="
  }
}

# Get next page using cursor
GET /api/posts?limit=20&cursor=eyJpZCI6MTAyMCwiY3JlYXRlZF9hdCI6IjIwMjUtMDgtMjdUMTE6MDA6MDBaIn0=

Response: 200 OK
{
  "data": [
    {
      "id": 1021,
      "title": "Next Post",
      "created_at": "2025-08-27T11:05:00Z",
      "cursor": "eyJpZCI6MTAyMSwiY3JlYXRlZF9hdCI6IjIwMjUtMDgtMjdUMTE6MDU6MDBaIn0="
    }
    // ... 19 more posts
  ],
  "pagination": {
    "limit": 20,
    "has_next": true,
    "has_prev": true,
    "next_cursor": "eyJpZCI6MTA0MCwiY3JlYXRlZF9hdCI6IjIwMjUtMDgtMjdUMTI6MDA6MDBaIn0=",
    "prev_cursor": "eyJpZCI6MTAwMCwiY3JlYXRlZF9hdCI6IjIwMjUtMDgtMjdUMDk6NTU6MDBaIn0="
  }
}
```

**Cursor Encoding Example:**
```javascript
// Cursor contains: {id: 1020, created_at: "2025-08-27T11:00:00Z"}
const cursor = {
  id: 1020,
  created_at: "2025-08-27T11:00:00Z"
};
const encodedCursor = btoa(JSON.stringify(cursor));
// Result: "eyJpZCI6MTAyMCwiY3JlYXRlZF9hdCI6IjIwMjUtMDgtMjdUMTE6MDA6MDBaIn0="
```

**Advantages:**
- Consistent performance regardless of position
- Handles real-time data changes gracefully
- No duplicate or missing items during pagination
- Efficient database queries

**Disadvantages:**
- Can't jump to arbitrary pages
- More complex to implement
- Opaque cursors harder to debug

### 4. Keyset Pagination (Seek Method)

Uses the values of ordered columns as pagination keys.

```http
# Get first page (no parameters needed)
GET /api/orders?limit=25&sort=created_at

Response: 200 OK
{
  "data": [
    {
      "id": 1001,
      "total": 150.00,
      "created_at": "2025-08-27T08:00:00Z"
    },
    {
      "id": 1002,
      "total": 200.00, 
      "created_at": "2025-08-27T08:15:00Z"
    }
    // ... 23 more orders
  ],
  "pagination": {
    "limit": 25,
    "has_next": true,
    "last_key": {
      "created_at": "2025-08-27T10:30:00Z",
      "id": 1025
    }
  },
  "links": {
    "self": "/api/orders?limit=25&sort=created_at",
    "next": "/api/orders?limit=25&sort=created_at&after_created_at=2025-08-27T10:30:00Z&after_id=1025"
  }
}

# Get next page using keyset
GET /api/orders?limit=25&sort=created_at&after_created_at=2025-08-27T10:30:00Z&after_id=1025

Response: 200 OK
{
  "data": [
    {
      "id": 1026,
      "total": 175.00,
      "created_at": "2025-08-27T10:35:00Z"
    }
    // ... 24 more orders
  ],
  "pagination": {
    "limit": 25,
    "has_next": true,
    "has_prev": true,
    "last_key": {
      "created_at": "2025-08-27T12:00:00Z",
      "id": 1050
    },
    "first_key": {
      "created_at": "2025-08-27T10:35:00Z", 
      "id": 1026
    }
  }
}
```

### 5. Time-Based Pagination

Specialized for time-series data using timestamps.

```http
# Get recent events
GET /api/events?since=2025-08-27T00:00:00Z&limit=50

Response: 200 OK
{
  "data": [
    {
      "id": 5001,
      "type": "user_login",
      "timestamp": "2025-08-27T10:30:00Z",
      "user_id": 123
    },
    {
      "id": 5002,
      "type": "order_created",
      "timestamp": "2025-08-27T10:32:00Z",
      "order_id": 456
    }
    // ... 48 more events
  ],
  "pagination": {
    "limit": 50,
    "since": "2025-08-27T00:00:00Z",
    "until": "2025-08-27T12:00:00Z",
    "oldest": "2025-08-27T10:30:00Z",
    "newest": "2025-08-27T12:00:00Z",
    "has_more": true
  },
  "links": {
    "self": "/api/events?since=2025-08-27T00:00:00Z&limit=50",
    "next": "/api/events?since=2025-08-27T12:00:00Z&limit=50"
  }
}
```

## Pagination Headers

### HTTP Headers for Pagination
```http
# Using Link header (RFC 5988)
GET /api/users?page=5&limit=20

Response: 200 OK
Link: <https://api.example.com/users?page=1&limit=20>; rel="first",
      <https://api.example.com/users?page=4&limit=20>; rel="prev", 
      <https://api.example.com/users?page=6&limit=20>; rel="next",
      <https://api.example.com/users?page=75&limit=20>; rel="last"
X-Total-Count: 1500
X-Page: 5
X-Per-Page: 20
X-Total-Pages: 75

# GitHub-style pagination headers
Link: <https://api.example.com/users?page=6&limit=20>; rel="next",
      <https://api.example.com/users?page=75&limit=20>; rel="last",
      <https://api.example.com/users?page=1&limit=20>; rel="first",
      <https://api.example.com/users?page=4&limit=20>; rel="prev"
X-RateLimit-Limit: 5000
X-RateLimit-Remaining: 4999
```

### Custom Pagination Headers
```http
Response: 200 OK
X-Pagination-Page: 5
X-Pagination-Limit: 20
X-Pagination-Total: 1500
X-Pagination-Pages: 75
X-Pagination-Has-Next: true
X-Pagination-Has-Prev: true
```

## Advanced Pagination Features

### Filtering with Pagination
```http
# Paginated filtered results
GET /api/products?category=electronics&price_min=100&price_max=500&page=2&limit=25

Response: 200 OK
{
  "data": [...],
  "pagination": {
    "page": 2,
    "limit": 25,
    "total": 150,
    "pages": 6,
    "has_next": true,
    "has_prev": true
  },
  "filters": {
    "category": "electronics",
    "price_min": 100,
    "price_max": 500
  },
  "links": {
    "self": "/api/products?category=electronics&price_min=100&price_max=500&page=2&limit=25",
    "next": "/api/products?category=electronics&price_min=100&price_max=500&page=3&limit=25",
    "prev": "/api/products?category=electronics&price_min=100&price_max=500&page=1&limit=25"
  }
}
```

### Sorting with Pagination
```http
# Paginated sorted results
GET /api/orders?sort=-created_at,total&page=1&limit=20

Response: 200 OK
{
  "data": [
    {
      "id": 1001,
      "total": 500.00,
      "created_at": "2025-08-27T12:00:00Z"
    }
    // ... sorted by newest first, then by total
  ],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 800,
    "pages": 40
  },
  "sort": [
    {"field": "created_at", "direction": "desc"},
    {"field": "total", "direction": "asc"}
  ]
}
```

### Search with Pagination
```http
# Paginated search results
GET /api/products/search?q=wireless+headphones&page=1&limit=15

Response: 200 OK
{
  "data": [
    {
      "id": 2001,
      "name": "Wireless Bluetooth Headphones",
      "score": 0.95,
      "highlights": {
        "name": "<em>Wireless</em> Bluetooth <em>Headphones</em>",
        "description": "High-quality <em>wireless</em> audio experience"
      }
    }
    // ... more search results
  ],
  "pagination": {
    "page": 1, 
    "limit": 15,
    "total": 47,
    "pages": 4
  },
  "search": {
    "query": "wireless headphones",
    "took_ms": 23,
    "total_matches": 47
  }
}
```

### Faceted Pagination
```http
# Paginated results with facets
GET /api/products?category=electronics&page=1&limit=20&include_facets=true

Response: 200 OK
{
  "data": [...],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 324,
    "pages": 17
  },
  "facets": {
    "brand": [
      {"name": "Apple", "count": 45},
      {"name": "Samsung", "count": 38},
      {"name": "Sony", "count": 29}
    ],
    "price_range": [
      {"range": "0-100", "count": 89},
      {"range": "100-500", "count": 156},
      {"range": "500+", "count": 79}
    ],
    "rating": [
      {"stars": 5, "count": 98},
      {"stars": 4, "count": 145},
      {"stars": 3, "count": 67}
    ]
  }
}
```

## Pagination Performance Optimization

### Database Query Optimization
```sql
-- Inefficient: Large offset
SELECT * FROM users ORDER BY created_at LIMIT 20 OFFSET 100000;

-- Efficient: Cursor-based with index
SELECT * FROM users 
WHERE created_at > '2025-08-27T10:00:00Z' 
   OR (created_at = '2025-08-27T10:00:00Z' AND id > 1000)
ORDER BY created_at, id 
LIMIT 20;

-- Efficient: Keyset pagination
SELECT * FROM products 
WHERE (price > 100) OR (price = 100 AND id > 5000)
ORDER BY price, id 
LIMIT 25;
```

### Caching Strategies
```http
# Cache paginated responses
GET /api/products?page=1&limit=20

Response: 200 OK
Cache-Control: public, max-age=300, stale-while-revalidate=600
ETag: "products-page-1-limit-20-v1"
Vary: Accept, Accept-Language

# Conditional request for cached page
GET /api/products?page=1&limit=20
If-None-Match: "products-page-1-limit-20-v1"

Response: 304 Not Modified
Cache-Control: public, max-age=300, stale-while-revalidate=600
ETag: "products-page-1-limit-20-v1"
```

### Count Optimization
```http
# Avoid expensive COUNT queries for large datasets
GET /api/users?page=1&limit=20&include_total=false

Response: 200 OK
{
  "data": [...],
  "pagination": {
    "page": 1,
    "limit": 20,
    "has_next": true,
    "has_prev": false,
    "estimated_total": "10000+",
    "exact_total_available": false
  }
}

# Provide total count only when specifically requested
GET /api/users?page=1&limit=20&include_total=true

Response: 200 OK
{
  "data": [...],
  "pagination": {
    "page": 1,
    "limit": 20, 
    "total": 15467,
    "pages": 774,
    "has_next": true,
    "has_prev": false
  },
  "warning": "Total count calculation may impact performance"
}
```

## Client-Side Pagination Handling

### JavaScript Pagination Client
```javascript
class PaginationClient {
  constructor(apiUrl) {
    this.apiUrl = apiUrl;
    this.cache = new Map();
  }

  async getPage(page, limit = 20, filters = {}) {
    const cacheKey = this.buildCacheKey(page, limit, filters);
    
    if (this.cache.has(cacheKey)) {
      return this.cache.get(cacheKey);
    }

    const params = new URLSearchParams({
      page: page.toString(),
      limit: limit.toString(),
      ...filters
    });

    const response = await fetch(`${this.apiUrl}?${params}`);
    const data = await response.json();
    
    this.cache.set(cacheKey, data);
    return data;
  }

  async getAllPages(limit = 20, filters = {}) {
    const allData = [];
    let page = 1;
    let hasNext = true;

    while (hasNext) {
      const response = await this.getPage(page, limit, filters);
      allData.push(...response.data);
      
      hasNext = response.pagination.has_next;
      page++;
    }

    return allData;
  }

  async getCursorPage(cursor = null, limit = 20) {
    const params = new URLSearchParams({
      limit: limit.toString()
    });
    
    if (cursor) {
      params.set('cursor', cursor);
    }

    const response = await fetch(`${this.apiUrl}?${params}`);
    return await response.json();
  }

  buildCacheKey(page, limit, filters) {
    return `page:${page}-limit:${limit}-filters:${JSON.stringify(filters)}`;
  }
}

// Usage
const client = new PaginationClient('/api/products');

// Get specific page
const page1 = await client.getPage(1, 25, {category: 'electronics'});

// Get all data (use carefully with large datasets)
const allProducts = await client.getAllPages(50);

// Cursor-based pagination
const firstPage = await client.getCursorPage();
const nextPage = await client.getCursorPage(firstPage.pagination.next_cursor);
```

### Infinite Scroll Implementation
```javascript
class InfiniteScrollPagination {
  constructor(container, apiUrl) {
    this.container = container;
    this.apiUrl = apiUrl;
    this.loading = false;
    this.hasMore = true;
    this.nextCursor = null;
    
    this.setupScrollListener();
  }

  setupScrollListener() {
    window.addEventListener('scroll', () => {
      if (this.shouldLoadMore()) {
        this.loadNext();
      }
    });
  }

  shouldLoadMore() {
    const scrollTop = window.pageYOffset;
    const windowHeight = window.innerHeight;
    const docHeight = document.documentElement.scrollHeight;
    
    return (scrollTop + windowHeight >= docHeight - 1000) && 
           !this.loading && 
           this.hasMore;
  }

  async loadNext() {
    if (this.loading || !this.hasMore) return;
    
    this.loading = true;
    this.showLoader();

    try {
      const params = new URLSearchParams({
        limit: '20'
      });
      
      if (this.nextCursor) {
        params.set('cursor', this.nextCursor);
      }

      const response = await fetch(`${this.apiUrl}?${params}`);
      const data = await response.json();
      
      this.appendItems(data.data);
      this.nextCursor = data.pagination.next_cursor;
      this.hasMore = data.pagination.has_next;
      
    } catch (error) {
      this.showError(error);
    } finally {
      this.loading = false;
      this.hideLoader();
    }
  }

  appendItems(items) {
    items.forEach(item => {
      const element = this.createItemElement(item);
      this.container.appendChild(element);
    });
  }

  createItemElement(item) {
    const div = document.createElement('div');
    div.className = 'item';
    div.innerHTML = `
      <h3>${item.name}</h3>
      <p>${item.description}</p>
    `;
    return div;
  }

  showLoader() {
    const loader = document.getElementById('loader');
    if (loader) loader.style.display = 'block';
  }

  hideLoader() {
    const loader = document.getElementById('loader');
    if (loader) loader.style.display = 'none';
  }

  showError(error) {
    console.error('Failed to load more items:', error);
    // Show user-friendly error message
  }
}
```

## Pagination Best Practices

### 1. Provide Multiple Pagination Options
```http
# Support both offset and cursor pagination
GET /api/posts?page=5&limit=20          # Offset-based
GET /api/posts?cursor=abc123&limit=20   # Cursor-based

# Allow clients to choose pagination style
GET /api/data?pagination_type=offset&page=1&limit=50
GET /api/data?pagination_type=cursor&cursor=xyz&limit=50
```

### 2. Reasonable Default and Maximum Limits
```http
# Default limit if not specified
GET /api/users
# Returns 25 items by default

# Enforce maximum limits
GET /api/users?limit=10000

Response: 400 Bad Request
{
  "error": "invalid_limit",
  "message": "Limit cannot exceed 100",
  "max_limit": 100,
  "requested_limit": 10000
}
```

### 3. Consistent Response Format
```json
{
  "data": [],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 1500,
    "pages": 75,
    "has_next": true,
    "has_prev": false
  },
  "links": {
    "self": "...",
    "first": "...",
    "last": "...",
    "next": "...",
    "prev": "..."
  },
  "meta": {
    "timestamp": "2025-08-27T10:30:00Z",
    "took_ms": 45
  }
}
```

### 4. Handle Edge Cases
```http
# Empty results
GET /api/users?page=999&limit=20

Response: 200 OK
{
  "data": [],
  "pagination": {
    "page": 999,
    "limit": 20,
    "total": 1500,
    "pages": 75,
    "has_next": false,
    "has_prev": true
  },
  "message": "No results found for page 999"
}

# Invalid page number
GET /api/users?page=0&limit=20

Response: 400 Bad Request
{
  "error": "invalid_page",
  "message": "Page number must be greater than 0",
  "valid_range": "1-75"
}
```

## Interview Questions

1. **Q: What are the main differences between offset-based and cursor-based pagination?**
   **A:** Offset-based uses page numbers or offsets, is intuitive but has performance issues with large datasets and consistency problems with changing data. Cursor-based uses opaque tokens, provides consistent performance and handles real-time changes well, but can't jump to arbitrary pages.

2. **Q: Why does offset-based pagination become slow with large offsets?**
   **A:** Databases must scan and skip all rows up to the offset before returning results. For example, OFFSET 100000 requires scanning 100,000 rows. This becomes exponentially slower as offset increases, especially without proper indexing.

3. **Q: How do you handle pagination when the underlying data changes?**
   **A:** Use cursor-based pagination for consistency. With offset-based pagination, items can be duplicated or missed if data is inserted/deleted during pagination. Cursor-based pagination maintains position relative to actual data state.

4. **Q: What's the best way to implement pagination for search results?**
   **A:** Use cursor-based pagination with search relevance scores. Include stable sorting criteria (like document ID) as secondary sort to ensure consistent ordering. Consider caching search results for subsequent pages if search is expensive.

5. **Q: How do you optimize database queries for paginated results?**
   **A:** Use appropriate indexes on sort columns, avoid expensive COUNT queries for large datasets, implement keyset pagination for better performance, use LIMIT clauses, and consider read replicas for read-heavy pagination workloads.

6. **Q: What pagination strategy works best for real-time data streams?**
   **A:** Time-based pagination using timestamps works well for real-time data. Use 'since' parameters to get new data and implement polling or websockets for live updates. Cursor-based pagination also handles real-time changes gracefully.

7. **Q: How should you handle pagination in mobile applications?**
   **A:** Use smaller page sizes (10-20 items), implement infinite scroll for better UX, prefetch next page for smooth scrolling, cache previous pages for back navigation, and consider network conditions for adaptive page sizes.

8. **Q: What are the security considerations for pagination?**
   **A:** Validate and limit page sizes to prevent DoS attacks, implement rate limiting on pagination endpoints, ensure cursors are tamper-proof (signed/encrypted), validate user permissions for each page, and prevent information leakage through pagination metadata.

9. **Q: How do you implement pagination for aggregated or grouped data?**
   **A:** Paginate at the group level first, then within groups if needed. Include group metadata in pagination response, use nested pagination for large groups, and consider separate endpoints for group summaries vs detailed group contents.

10. **Q: What's the best approach for paginating across multiple data sources?**
    **A:** Use a coordination layer that merges results from multiple sources, implement cursor-based pagination with source identification in cursors, handle different data source response times, and consider eventual consistency issues when aggregating real-time data from multiple sources.
