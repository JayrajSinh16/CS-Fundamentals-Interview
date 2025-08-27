# API Versioning Strategies

API versioning is essential for maintaining backward compatibility while allowing APIs to evolve. It enables developers to introduce new features, fix bugs, and improve designs without breaking existing clients. Understanding different versioning strategies and their trade-offs is crucial for building maintainable APIs.

## Why API Versioning is Important

### Business Continuity
- **Client Compatibility**: Existing clients continue working during API updates
- **Gradual Migration**: Clients can migrate to new versions at their own pace
- **Risk Reduction**: Changes are isolated to specific versions
- **Service Level Agreements**: Meet contractual obligations for API stability

### Technical Benefits
- **Feature Evolution**: Add new functionality without breaking existing features
- **Bug Fixes**: Fix issues in newer versions while maintaining old behavior
- **Performance Improvements**: Optimize APIs without affecting legacy clients
- **Security Updates**: Apply security patches to specific versions

## Versioning Strategies

### 1. URL Path Versioning

Most common and visible versioning method using the URL path.

```http
# Version 1 API
GET /api/v1/users/123
Response: 200 OK
{
  "id": 123,
  "name": "John Doe",
  "email": "john@example.com"
}

# Version 2 API with expanded user data
GET /api/v2/users/123
Response: 200 OK
{
  "id": 123,
  "full_name": "John Doe",
  "email_address": "john@example.com",
  "profile": {
    "created_at": "2025-01-15T10:30:00Z",
    "last_login": "2025-08-27T09:15:00Z",
    "status": "active"
  },
  "preferences": {
    "timezone": "America/New_York",
    "language": "en-US"
  }
}

# Version 3 API with different response structure
GET /api/v3/users/123
Response: 200 OK
{
  "user": {
    "id": 123,
    "personal_info": {
      "full_name": "John Doe",
      "email_address": "john@example.com"
    },
    "account_info": {
      "created_at": "2025-01-15T10:30:00Z",
      "last_login": "2025-08-27T09:15:00Z",
      "status": "active"
    },
    "settings": {
      "timezone": "America/New_York",
      "language": "en-US"
    }
  },
  "meta": {
    "version": "3.0",
    "deprecated_fields": []
  }
}

# Different versions can have different endpoints
GET /api/v1/user-posts/123    # v1 naming
GET /api/v2/users/123/posts   # v2 improved naming
GET /api/v3/users/123/posts   # v3 same as v2 for this endpoint
```

**Advantages:**
- Clear and visible versioning
- Easy to implement and understand
- Simple routing and documentation
- Clear separation between versions

**Disadvantages:**
- URL changes for every version
- Can lead to URL proliferation
- Not RESTful (resource URLs change)
- Harder to maintain multiple versions

### 2. Header Versioning

Version specified in request headers, keeping URLs clean.

```http
# Default version (latest or v1)
GET /api/users/123
Response: 200 OK
X-API-Version: 1.0
{
  "id": 123,
  "name": "John Doe",
  "email": "john@example.com"
}

# Specific version via header
GET /api/users/123
X-API-Version: 2.0
Response: 200 OK
X-API-Version: 2.0
{
  "id": 123,
  "full_name": "John Doe",
  "email_address": "john@example.com",
  "profile": {...}
}

# Accept header versioning
GET /api/users/123
Accept: application/vnd.myapi.v2+json
Response: 200 OK
Content-Type: application/vnd.myapi.v2+json
{
  "id": 123,
  "full_name": "John Doe",
  "email_address": "john@example.com"
}

# Version not supported
GET /api/users/123
X-API-Version: 5.0
Response: 400 Bad Request
{
  "error": "unsupported_version",
  "message": "API version 5.0 is not supported",
  "supported_versions": ["1.0", "2.0", "3.0"],
  "latest_version": "3.0",
  "deprecation_info": {
    "v1.0": {
      "deprecated": true,
      "sunset_date": "2025-12-31"
    }
  }
}
```

**Advantages:**
- Clean, unchanging URLs
- More RESTful approach
- Easier to cache (with Vary header)
- Follows HTTP standards

**Disadvantages:**
- Less visible versioning
- Harder to test manually
- Requires client header support
- May be overlooked in documentation

### 3. Query Parameter Versioning

Version specified as a query parameter.

```http
# Default version (no parameter)
GET /api/users/123
Response: 200 OK
X-API-Version: 1.0

# Specific version via query parameter
GET /api/users/123?version=2
Response: 200 OK
X-API-Version: 2.0
{
  "id": 123,
  "full_name": "John Doe",
  "email_address": "john@example.com"
}

# Alternative parameter name
GET /api/users/123?api_version=3.0
Response: 200 OK
X-API-Version: 3.0

# Version with other parameters
GET /api/users?version=2&status=active&limit=20
Response: 200 OK
{
  "users": [...],
  "pagination": {...},
  "version_info": {
    "requested": "2.0",
    "served": "2.0"
  }
}
```

**Advantages:**
- Easy to implement and test
- Visible in URLs
- Works with all HTTP clients
- Can be easily added to existing APIs

**Disadvantages:**
- URLs become version-specific
- Can clutter query strings
- May conflict with other parameters
- Less clean than header approach

### 4. Content Negotiation Versioning

Use Accept header with vendor-specific media types.

```http
# GitHub-style versioning
GET /api/users/123
Accept: application/vnd.github.v3+json
Response: 200 OK
Content-Type: application/vnd.github.v3+json

# Custom vendor media types
GET /api/products/456
Accept: application/vnd.mycompany.ecommerce.v2+json
Response: 200 OK
Content-Type: application/vnd.mycompany.ecommerce.v2+json

# Multiple format support
GET /api/orders/789
Accept: application/vnd.myapi.v2+xml
Response: 200 OK
Content-Type: application/vnd.myapi.v2+xml
<?xml version="1.0"?>
<order>
  <id>789</id>
  <total>150.00</total>
</order>

# Fallback when version not available
GET /api/users/123
Accept: application/vnd.myapi.v5+json, application/vnd.myapi.v3+json
Response: 200 OK
Content-Type: application/vnd.myapi.v3+json
X-Served-Version: v3
X-Requested-Version: v5
{
  "message": "Requested version v5 not available, serving v3"
}
```

**Advantages:**
- Follows HTTP standards closely
- Supports multiple formats per version
- Clean URLs
- Proper content negotiation

**Disadvantages:**
- Complex Accept header syntax
- Harder to test manually
- May be overkill for simple APIs
- Requires client sophistication

### 5. Subdomain Versioning

Different versions hosted on different subdomains.

```http
# Version 1
GET https://v1.api.example.com/users/123
Response: 200 OK
{
  "id": 123,
  "name": "John Doe"
}

# Version 2
GET https://v2.api.example.com/users/123
Response: 200 OK
{
  "id": 123,
  "full_name": "John Doe"
}

# Latest version alias
GET https://api.example.com/users/123
# Redirects to latest version
Response: 302 Found
Location: https://v3.api.example.com/users/123

# Version-specific features
GET https://v2.api.example.com/analytics/dashboard
# Only available in v2+
```

**Advantages:**
- Complete separation of versions
- Independent scaling and deployment
- Clear version identification
- Can use different infrastructure

**Disadvantages:**
- DNS and SSL management overhead
- Harder to share code between versions
- More complex infrastructure
- Potential CORS complications

## Version Management Strategies

### 1. Semantic Versioning

Use semantic versioning (MAJOR.MINOR.PATCH) for API versions.

```http
# Breaking changes increment major version
GET /api/v1.0.0/users/123      # Initial version
GET /api/v2.0.0/users/123      # Breaking changes (field name changes)

# New features increment minor version
GET /api/v2.1.0/users/123      # Added new optional fields
GET /api/v2.2.0/users/123      # Added new endpoint

# Bug fixes increment patch version
GET /api/v2.2.1/users/123      # Fixed calculation error
GET /api/v2.2.2/users/123      # Fixed response format issue

# Version negotiation
GET /api/users/123
Accept: application/vnd.myapi.v2.1+json
Response: 200 OK
Content-Type: application/vnd.myapi.v2.2+json
X-Actual-Version: 2.2.2
X-Requested-Version: 2.1.0
{
  "note": "Serving compatible version 2.2.2 for request 2.1.0"
}
```

### 2. Date-Based Versioning

Use dates for version identification (common in public APIs).

```http
# Stripe-style date versioning
GET /api/charges/ch_123
Stripe-Version: 2025-08-27
Response: 200 OK
{
  "id": "ch_123",
  "amount": 5000,
  "currency": "usd",
  "api_version": "2025-08-27"
}

# GitHub-style date versioning  
GET /api/repos/owner/repo
Accept: application/vnd.github+json
X-GitHub-Api-Version: 2025-08-27
Response: 200 OK

# Default to latest version if none specified
GET /api/payments/pay_123
Response: 200 OK
X-API-Version: 2025-08-27
X-Version-Type: latest
```

### 3. Feature Flags and Gradual Rollouts

Use feature flags to enable new functionality gradually.

```http
# Request with feature flags
GET /api/users/123
X-Feature-Flags: new-profile-format,enhanced-security
Response: 200 OK
X-Enabled-Features: new-profile-format,enhanced-security
{
  "id": 123,
  "profile": {
    "new_format": true,
    "enhanced_data": {...}
  },
  "security": {
    "mfa_enabled": true,
    "last_security_check": "2025-08-27T10:30:00Z"
  }
}

# Gradual rollout based on user segments
GET /api/dashboard
Authorization: Bearer user_token
Response: 200 OK
X-User-Segment: beta
X-Feature-Version: dashboard-v2
{
  "layout": "v2",
  "features": ["advanced-analytics", "real-time-updates"],
  "beta_disclaimer": "You're using our new dashboard beta"
}
```

## Backward Compatibility Strategies

### 1. Additive Changes Only

Add new fields and endpoints without removing or changing existing ones.

```http
# Version 1.0 - Original response
GET /api/v1/products/123
Response: 200 OK
{
  "id": 123,
  "name": "Product Name",
  "price": 99.99
}

# Version 1.1 - Added fields (backward compatible)
GET /api/v1.1/products/123
Response: 200 OK
{
  "id": 123,
  "name": "Product Name",
  "price": 99.99,
  "description": "Product description",    # New field
  "category": "Electronics",              # New field
  "ratings": {                           # New nested object
    "average": 4.5,
    "count": 127
  }
}

# Version 1.2 - More new fields
GET /api/v1.2/products/123
Response: 200 OK
{
  "id": 123,
  "name": "Product Name", 
  "price": 99.99,
  "description": "Product description",
  "category": "Electronics",
  "ratings": {
    "average": 4.5,
    "count": 127
  },
  "inventory": {                         # New in v1.2
    "stock": 15,
    "warehouse": "West Coast"
  },
  "shipping_info": {                     # New in v1.2
    "weight": "1.2kg",
    "dimensions": "10x8x3cm"
  }
}
```

### 2. Deprecation Warnings

Provide clear warnings about deprecated features.

```http
# Deprecated field usage
GET /api/v2/users/123
Response: 200 OK
X-Deprecated-Fields: user_name
Warning: 299 - "Field 'user_name' is deprecated, use 'full_name' instead"
{
  "id": 123,
  "user_name": "John Doe",              # Deprecated
  "full_name": "John Doe",              # Preferred
  "email": "john@example.com",
  "deprecation_notices": [
    {
      "field": "user_name",
      "replacement": "full_name",
      "sunset_date": "2025-12-31",
      "documentation": "https://docs.api.com/migration/user-name"
    }
  ]
}

# Deprecated endpoint
GET /api/v1/user-list
Response: 200 OK
Warning: 299 - "Endpoint deprecated, use /api/v2/users instead"
Sunset: Wed, 31 Dec 2025 23:59:59 GMT
Link: </api/v2/users>; rel="successor-version"
{
  "users": [...],
  "deprecation_warning": {
    "message": "This endpoint will be removed on 2025-12-31",
    "alternative": "/api/v2/users",
    "migration_guide": "https://docs.api.com/migration/v1-to-v2"
  }
}
```

### 3. Field Aliasing

Support both old and new field names during transition periods.

```http
# API supports both old and new field names
GET /api/v2/orders/456
Response: 200 OK
{
  "id": 456,
  "order_total": 150.00,               # New field name
  "total": 150.00,                     # Alias for old clients
  "customer_info": {                   # New structure
    "name": "John Doe",
    "email": "john@example.com"
  },
  "customer": "John Doe",              # Alias for old clients
  "field_mappings": {
    "total": "order_total",
    "customer": "customer_info.name"
  }
}

# POST/PUT requests accept both formats
POST /api/v2/orders
Content-Type: application/json
{
  "total": 200.00,                     # Old field name accepted
  "customer": "Jane Smith"             # Old field name accepted
}

Response: 201 Created
{
  "id": 457,
  "order_total": 200.00,               # Response uses new format
  "customer_info": {
    "name": "Jane Smith"
  },
  "created_from_legacy_format": true
}
```

## Version Lifecycle Management

### 1. Version Release Process

```http
# Alpha version (internal testing)
GET /api/alpha/v3/users/123
X-API-Stage: alpha
Response: 200 OK
X-Version-Stage: alpha
X-Stability: experimental
{
  "warning": "Alpha version - subject to breaking changes",
  "data": {...}
}

# Beta version (external testing)
GET /api/beta/v3/users/123
X-API-Stage: beta
Response: 200 OK
X-Version-Stage: beta
X-Stability: preview
{
  "notice": "Beta version - feature complete but may have bugs",
  "data": {...}
}

# Production version
GET /api/v3/users/123
Response: 200 OK
X-Version-Stage: production
X-Stability: stable
{
  "data": {...}
}

# Version status endpoint
GET /api/versions
Response: 200 OK
{
  "versions": [
    {
      "version": "1.0",
      "status": "deprecated",
      "sunset_date": "2025-12-31",
      "documentation": "/docs/v1"
    },
    {
      "version": "2.0", 
      "status": "maintenance",
      "end_of_life": "2026-06-30",
      "documentation": "/docs/v2"
    },
    {
      "version": "3.0",
      "status": "current",
      "documentation": "/docs/v3"
    },
    {
      "version": "4.0",
      "status": "beta",
      "expected_release": "2025-12-01",
      "documentation": "/docs/v4-beta"
    }
  ],
  "recommended_version": "3.0",
  "minimum_supported": "2.0"
}
```

### 2. Sunset and Deprecation Process

```http
# Deprecation announcement
GET /api/v1/users/123
Response: 200 OK
Deprecation: true
Sunset: Wed, 31 Dec 2025 23:59:59 GMT
Link: </docs/migration/v1-to-v3>; rel="deprecation"
X-Deprecation-Notice: "API v1 will be sunset on 2025-12-31"
{
  "data": {...},
  "deprecation_info": {
    "deprecated": true,
    "sunset_date": "2025-12-31T23:59:59Z",
    "replacement_version": "3.0",
    "migration_guide": "https://docs.api.com/migration/v1-to-v3",
    "support_contact": "api-support@example.com"
  }
}

# Approaching sunset date
GET /api/v1/users/123
Response: 200 OK
Warning: 299 - "API v1 will be discontinued in 30 days"
Sunset: Wed, 31 Dec 2025 23:59:59 GMT
{
  "data": {...},
  "urgent_notice": {
    "message": "This API version will stop working in 30 days",
    "action_required": true,
    "migration_deadline": "2025-12-31T23:59:59Z"
  }
}

# After sunset date
GET /api/v1/users/123
Response: 410 Gone
{
  "error": "version_sunset",
  "message": "API version 1.0 has been discontinued",
  "sunset_date": "2025-12-31T23:59:59Z",
  "alternatives": [
    {
      "version": "3.0",
      "url": "/api/v3/users/123",
      "migration_guide": "https://docs.api.com/migration/v1-to-v3"
    }
  ]
}
```

## Version Documentation and Discovery

### 1. API Discovery

```http
# Root endpoint shows available versions
GET /api/
Response: 200 OK
{
  "service_name": "Example API",
  "description": "RESTful API for example application",
  "versions": {
    "v1": {
      "status": "deprecated",
      "base_url": "/api/v1",
      "documentation": "/docs/v1",
      "sunset_date": "2025-12-31"
    },
    "v2": {
      "status": "supported", 
      "base_url": "/api/v2",
      "documentation": "/docs/v2"
    },
    "v3": {
      "status": "current",
      "base_url": "/api/v3", 
      "documentation": "/docs/v3"
    }
  },
  "latest_version": "v3",
  "default_version": "v3"
}

# Version-specific capability discovery
GET /api/v3/
Response: 200 OK
{
  "version": "3.0.0",
  "resources": [
    {"name": "users", "url": "/api/v3/users"},
    {"name": "products", "url": "/api/v3/products"},
    {"name": "orders", "url": "/api/v3/orders"}
  ],
  "features": [
    "pagination",
    "filtering",
    "real_time_updates",
    "webhook_support"
  ],
  "authentication": ["bearer_token", "api_key"],
  "documentation": "/docs/v3"
}
```

### 2. OpenAPI Specification Versioning

```yaml
# openapi-v3.yaml
openapi: 3.0.0
info:
  title: Example API
  version: 3.0.0
  description: Version 3 of the Example API
  contact:
    name: API Support
    url: https://example.com/support
    email: api-support@example.com
  license:
    name: MIT
    url: https://opensource.org/licenses/MIT
  
servers:
  - url: https://api.example.com/v3
    description: Production server
  - url: https://staging-api.example.com/v3
    description: Staging server

paths:
  /users/{id}:
    get:
      summary: Get user by ID
      parameters:
        - name: id
          in: path
          required: true
          schema:
            type: integer
      responses:
        '200':
          description: User found
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/UserV3'
        '404':
          description: User not found

components:
  schemas:
    UserV3:
      type: object
      properties:
        id:
          type: integer
        full_name:
          type: string
        email_address:
          type: string
        profile:
          $ref: '#/components/schemas/UserProfile'
```

## Interview Questions

1. **Q: What are the main API versioning strategies and their trade-offs?**
   **A:** URL path versioning (clear but changes URLs), header versioning (clean URLs but less visible), query parameter versioning (easy to test but clutters URLs), content negotiation (RESTful but complex), and subdomain versioning (complete separation but infrastructure overhead).

2. **Q: How do you handle backward compatibility when evolving APIs?**
   **A:** Use additive changes only, maintain field aliases during transitions, provide deprecation warnings, implement gradual rollouts with feature flags, and maintain multiple version support with clear sunset timelines.

3. **Q: What's the difference between semantic versioning and date-based versioning for APIs?**
   **A:** Semantic versioning (MAJOR.MINOR.PATCH) indicates breaking changes vs new features vs bug fixes. Date-based versioning (2025-08-27) indicates when changes were made, common for public APIs with frequent updates like Stripe or GitHub.

4. **Q: How do you properly deprecate an API version?**
   **A:** Announce deprecation early with clear timeline, use HTTP headers (Deprecation, Sunset), provide migration guides, send warnings in responses, gradually reduce support, and finally return 410 Gone after sunset date.

5. **Q: What versioning strategy works best for microservices?**
   **A:** Service-level versioning with semantic versioning, contract-first API design, independent service versions, consumer-driven contracts, and careful coordination between service versions to avoid breaking dependencies.

6. **Q: How do you version APIs when clients have different update cycles?**
   **A:** Maintain multiple versions simultaneously, provide long deprecation periods, use feature flags for gradual rollouts, implement version negotiation, and offer automated migration tools where possible.

7. **Q: What's the recommended approach for versioning REST APIs in production?**
   **A:** Start with URL path versioning for simplicity, use semantic versioning for version numbers, provide comprehensive documentation, implement proper deprecation processes, and consider header-based versioning for more mature APIs.

8. **Q: How do you handle database schema changes across API versions?**
   **A:** Use database views for different API versions, implement data transformation layers, maintain backward-compatible schema changes, use event sourcing for version independence, and consider separate databases for major version differences.

9. **Q: What are the challenges of maintaining multiple API versions?**
   **A:** Code duplication and maintenance overhead, testing complexity across versions, documentation management, security patches across versions, performance monitoring per version, and coordinating client migrations.

10. **Q: How do you implement version negotiation in REST APIs?**
    **A:** Support version specification through multiple methods (header, query param, URL), implement fallback to supported versions, provide clear error messages for unsupported versions, and return version information in responses.
