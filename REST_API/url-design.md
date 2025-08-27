# URL Design & Resource Naming

Proper URL design is fundamental to creating intuitive and maintainable REST APIs. Well-designed URLs should be self-explanatory, hierarchical, and follow consistent patterns that make the API predictable for developers. This guide covers best practices for URL structure and resource naming conventions.

## URL Design Principles

### 1. Use Nouns, Not Verbs
URLs should represent resources (nouns), not actions (verbs). The HTTP method indicates the action.

```http
# Good - Resource-based URLs
GET    /api/users              # Get all users
POST   /api/users              # Create new user
GET    /api/users/123          # Get specific user
PUT    /api/users/123          # Update user
DELETE /api/users/123          # Delete user

# Bad - Action-based URLs
GET    /api/getUsers
POST   /api/createUser
GET    /api/getUserById/123
PUT    /api/updateUser/123
DELETE /api/deleteUser/123
```

### 2. Use Plural Nouns for Collections
Collections should use plural nouns to indicate they contain multiple resources.

```http
# Good - Plural nouns
GET /api/users                 # Collection of users
GET /api/products              # Collection of products
GET /api/orders                # Collection of orders

# Bad - Singular nouns
GET /api/user
GET /api/product
GET /api/order
```

### 3. Hierarchical Structure for Relationships
Use nested URLs to represent resource relationships and hierarchies.

```http
# User's posts
GET    /api/users/123/posts           # Get all posts by user 123
POST   /api/users/123/posts           # Create new post for user 123
GET    /api/users/123/posts/456       # Get specific post by user
PUT    /api/users/123/posts/456       # Update user's post
DELETE /api/users/123/posts/456       # Delete user's post

# Order items
GET    /api/orders/789/items          # Get all items in order
POST   /api/orders/789/items          # Add item to order
GET    /api/orders/789/items/101      # Get specific order item
PUT    /api/orders/789/items/101      # Update order item
DELETE /api/orders/789/items/101      # Remove item from order

# Company departments and employees
GET /api/companies/456/departments              # Company's departments
GET /api/companies/456/departments/789/employees # Department employees
```

### 4. Use Hyphens, Not Underscores
Use hyphens to separate words in URLs for better readability.

```http
# Good - Hyphens
GET /api/user-profiles
GET /api/order-items
GET /api/shipping-addresses

# Bad - Underscores
GET /api/user_profiles
GET /api/order_items
GET /api/shipping_addresses

# Bad - camelCase
GET /api/userProfiles
GET /api/orderItems
```

### 5. Use Lowercase Letters
URLs should be lowercase for consistency and to avoid case-sensitivity issues.

```http
# Good - Lowercase
GET /api/users/123/profile-settings
GET /api/products/categories

# Bad - Mixed case
GET /api/Users/123/ProfileSettings
GET /api/Products/Categories
GET /api/USERS/123/profile-settings
```

## Resource Naming Conventions

### Collections and Individual Resources
```http
# Collections (plural)
GET    /api/users              # All users
POST   /api/users              # Create user
GET    /api/products           # All products
GET    /api/orders             # All orders

# Individual resources (plural collection + ID)
GET    /api/users/123          # Specific user
PUT    /api/users/123          # Update user
DELETE /api/users/123          # Delete user
GET    /api/products/456       # Specific product
GET    /api/orders/789         # Specific order
```

### Sub-resources and Nested Collections
```http
# One-to-many relationships
GET /api/users/123/posts                    # User's posts
GET /api/users/123/comments                 # User's comments
GET /api/orders/456/items                   # Order items
GET /api/categories/789/products            # Products in category

# Many-to-many relationships
GET /api/users/123/roles                    # User's roles
GET /api/projects/456/contributors          # Project contributors
GET /api/courses/789/students               # Course enrollments

# Deeply nested resources (limit depth)
GET /api/companies/123/departments/456/employees/789
GET /api/users/123/projects/456/tasks/789/comments
```

### Action Resources for Non-CRUD Operations
When you need to perform actions that don't fit CRUD operations, model them as resources.

```http
# User actions
POST /api/users/123/activate               # Activate user account
POST /api/users/123/deactivate             # Deactivate user account
POST /api/users/123/reset-password         # Reset user password
POST /api/users/123/send-notification      # Send notification to user

# Order actions
POST /api/orders/456/cancel                # Cancel order
POST /api/orders/456/ship                  # Ship order
POST /api/orders/456/refund                # Process refund

# System actions
POST /api/system/backup                    # Create system backup
POST /api/system/maintenance-mode          # Enable maintenance mode
POST /api/cache/clear                      # Clear application cache

# Batch operations
POST /api/users/bulk-create                # Create multiple users
POST /api/orders/bulk-update               # Update multiple orders
DELETE /api/products/bulk-delete           # Delete multiple products
```

## URL Patterns and Examples

### Basic CRUD Patterns
```http
# Users resource
GET    /api/users                    # List all users
POST   /api/users                    # Create new user
GET    /api/users/123                # Get user by ID
PUT    /api/users/123                # Update entire user
PATCH  /api/users/123                # Partial user update
DELETE /api/users/123                # Delete user

# Products resource
GET    /api/products                 # List all products
POST   /api/products                 # Create new product
GET    /api/products/abc-123         # Get product by ID/slug
PUT    /api/products/abc-123         # Update product
DELETE /api/products/abc-123         # Delete product
```

### Filtering and Query Parameters
```http
# Filtering collections
GET /api/users?status=active                    # Active users only
GET /api/users?role=admin&status=active         # Active admin users
GET /api/products?category=electronics&in_stock=true

# Pagination
GET /api/users?page=2&limit=20                  # Page 2, 20 per page
GET /api/users?offset=40&limit=20               # Skip 40, take 20

# Sorting
GET /api/users?sort=created_at                  # Sort by creation date
GET /api/users?sort=-created_at                 # Reverse sort (newest first)
GET /api/users?sort=name,created_at             # Multiple sort fields

# Field selection
GET /api/users?fields=id,name,email             # Only specific fields
GET /api/users/123?include=posts,comments       # Include related data

# Search
GET /api/users?search=john                      # Search users
GET /api/products?q=laptop&category=electronics # Complex search
```

### Versioning in URLs
```http
# Version in URL path (common approach)
GET /api/v1/users/123
GET /api/v2/users/123

# Version in subdomain
GET https://v1.api.example.com/users/123
GET https://v2.api.example.com/users/123

# Version in query parameter (less common)
GET /api/users/123?version=1
GET /api/users/123?version=2

# Version in header (preferred)
GET /api/users/123
Accept: application/vnd.myapi.v1+json
```

### File and Media Resources
```http
# File uploads
POST /api/users/123/avatar              # Upload user avatar
POST /api/documents                     # Upload document
POST /api/products/456/images           # Upload product images

# File downloads
GET /api/users/123/avatar               # Download avatar
GET /api/documents/789/download         # Download document
GET /api/reports/456/pdf                # Download PDF report

# Media processing
POST /api/images/123/resize             # Resize image
POST /api/videos/456/convert            # Convert video format
GET  /api/images/123/thumbnails         # Get image thumbnails
```

## Advanced URL Patterns

### Matrix Parameters
```http
# Matrix parameters for complex filtering
GET /api/products;color=red;size=large;brand=nike
GET /api/flights;from=NYC;to=LAX;date=2025-08-27
```

### Resource Aliases and Shortcuts
```http
# Current user shortcut
GET /api/users/me                       # Current authenticated user
GET /api/users/me/profile              # Current user's profile
GET /api/users/me/notifications        # Current user's notifications

# Admin shortcuts
GET /api/admin/dashboard               # Admin dashboard data
GET /api/admin/reports                 # Admin reports
GET /api/admin/users/recent            # Recently registered users
```

### Composite Resources
```http
# Dashboard combining multiple resources
GET /api/dashboard                     # Overall dashboard
GET /api/users/123/dashboard          # User-specific dashboard

# Reports and analytics
GET /api/reports/sales                # Sales report
GET /api/reports/users/activity       # User activity report
GET /api/analytics/products/popular   # Popular products analytics
```

## URL Design Anti-patterns

### 1. Verbs in Resource URLs
```http
# Bad - Verbs in URLs
POST /api/createUser
GET  /api/getAllUsers
PUT  /api/updateUser/123
DELETE /api/deleteUser/123

# Good - Use HTTP methods for actions
POST   /api/users
GET    /api/users
PUT    /api/users/123
DELETE /api/users/123
```

### 2. Deep Nesting
```http
# Bad - Too deeply nested
GET /api/companies/123/departments/456/teams/789/members/101/projects/202

# Good - Limit nesting depth
GET /api/companies/123/departments
GET /api/departments/456/teams
GET /api/teams/789/members
GET /api/members/101/projects
```

### 3. Inconsistent Naming
```http
# Bad - Inconsistent patterns
GET /api/user                    # Singular
GET /api/products               # Plural
GET /api/getOrders              # Verb
GET /api/order_items            # Underscore

# Good - Consistent patterns
GET /api/users
GET /api/products
GET /api/orders
GET /api/order-items
```

### 4. Exposing Implementation Details
```http
# Bad - Implementation details
GET /api/database/users/table
GET /api/cache/products/123
GET /api/microservice/orders

# Good - Abstract interface
GET /api/users
GET /api/products/123
GET /api/orders
```

## Query Parameter Best Practices

### Standard Query Parameters
```http
# Pagination
?page=1&limit=20                # Page-based pagination
?offset=0&limit=20              # Offset-based pagination
?cursor=abc123&limit=20         # Cursor-based pagination

# Sorting
?sort=name                      # Ascending sort
?sort=-created_at               # Descending sort
?sort=name,-created_at          # Multiple fields

# Filtering
?status=active                  # Simple filter
?created_after=2025-01-01       # Date filter
?price_min=100&price_max=500    # Range filter

# Field selection
?fields=id,name,email           # Specific fields only
?include=posts,comments         # Include related data
?exclude=password,internal_id   # Exclude sensitive fields

# Search
?q=search+term                  # General search
?search=john&fields=name,email  # Search specific fields
```

### Complex Query Examples
```http
# E-commerce product search
GET /api/products?
    category=electronics&
    brand=apple&
    price_min=500&
    price_max=2000&
    in_stock=true&
    sort=-rating,price&
    page=1&
    limit=20&
    fields=id,name,price,rating,image_url

# User management with filters
GET /api/users?
    role=admin,moderator&
    status=active&
    created_after=2025-01-01&
    sort=-last_login&
    include=profile,permissions&
    page=2&
    limit=50

# Order reporting
GET /api/orders?
    status=completed&
    date_from=2025-08-01&
    date_to=2025-08-31&
    payment_method=credit_card&
    sort=-total_amount&
    fields=id,customer_name,total_amount,created_at
```

## URL Security Considerations

### 1. Avoid Sensitive Data in URLs
```http
# Bad - Sensitive data in URL
GET /api/users/reset-password?token=secret123&email=user@example.com

# Good - Sensitive data in request body
POST /api/users/reset-password
Content-Type: application/json
{
  "token": "secret123",
  "email": "user@example.com"
}
```

### 2. Validate and Sanitize Parameters
```http
# Validate ID parameters
GET /api/users/123                  # Valid numeric ID
GET /api/users/abc                  # Invalid - return 400

# Prevent path traversal
GET /api/files/../../../etc/passwd  # Block this pattern
GET /api/files/document.pdf         # Allow valid filenames
```

### 3. Use UUIDs for Sensitive Resources
```http
# Better - UUIDs harder to guess
GET /api/users/550e8400-e29b-41d4-a716-446655440000
GET /api/orders/6ba7b810-9dad-11d1-80b4-00c04fd430c8

# Avoid - Sequential IDs expose information
GET /api/users/1                    # Reveals there's only 1 user
GET /api/orders/12345              # Easy to guess other orders
```

## Documentation Examples

### API Documentation Format
```yaml
# OpenAPI/Swagger example
paths:
  /api/users:
    get:
      summary: List users
      parameters:
        - name: page
          in: query
          schema:
            type: integer
            default: 1
        - name: limit
          in: query
          schema:
            type: integer
            default: 20
    post:
      summary: Create user
      requestBody:
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/CreateUserRequest'
              
  /api/users/{userId}:
    get:
      summary: Get user by ID
      parameters:
        - name: userId
          in: path
          required: true
          schema:
            type: string
```

## Interview Questions

1. **Q: Why should REST URLs use nouns instead of verbs?**
   **A:** URLs represent resources (nouns), while HTTP methods represent actions (verbs). This separation makes the API more intuitive and follows REST principles. For example, `GET /api/users` is clearer than `GET /api/getUsers`.

2. **Q: How do you handle nested resources in URL design?**
   **A:** Use hierarchical URLs to show relationships: `/api/users/123/posts` for a user's posts. Limit nesting depth to 2-3 levels. For deeper relationships, use query parameters or separate endpoints with references.

3. **Q: What's the difference between using hyphens and underscores in URLs?**
   **A:** Hyphens are preferred because they're more readable and SEO-friendly. Search engines treat hyphens as word separators but underscores as word connectors. Example: `/api/user-profiles` is better than `/api/user_profiles`.

4. **Q: How should you handle actions that don't fit CRUD operations?**
   **A:** Model them as resources using POST to action endpoints: `POST /api/users/123/activate`, `POST /api/orders/456/cancel`. This maintains RESTful principles while supporting business actions.

5. **Q: What are the best practices for URL versioning?**
   **A:** Use version in URL path (`/api/v1/users`) for simplicity, or in Accept header (`Accept: application/vnd.myapi.v1+json`) for better REST compliance. Avoid query parameters for versioning.

6. **Q: How do you design URLs for filtering and searching?**
   **A:** Use query parameters: `?status=active&role=admin` for filtering, `?q=search+term` for search, `?sort=name,-created_at` for sorting. Keep parameters intuitive and well-documented.

7. **Q: Should you use singular or plural nouns for collections?**
   **A:** Always use plural nouns for collections (`/api/users`) to indicate they contain multiple resources. Individual resources use the same plural form with an ID (`/api/users/123`).

8. **Q: How do you handle file uploads and downloads in URL design?**
   **A:** Use resource-specific endpoints: `POST /api/users/123/avatar` for uploads, `GET /api/users/123/avatar` for downloads. For general files, use `/api/files` with appropriate sub-resources.

9. **Q: What security considerations apply to URL design?**
   **A:** Avoid sensitive data in URLs (use request body instead), validate all parameters, use UUIDs for sensitive resources instead of sequential IDs, and prevent path traversal attacks.

10. **Q: How do you design URLs for complex queries and reports?**
    **A:** Use query parameters for filters and options: `/api/reports/sales?date_from=2025-01-01&date_to=2025-12-31&format=pdf`. For very complex queries, consider POST endpoints with query data in the request body.
