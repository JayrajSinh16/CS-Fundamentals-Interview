# REST Principles & Constraints

REST (Representational State Transfer) is an architectural style defined by Roy Fielding in 2000. It provides a set of constraints that, when applied, create scalable, maintainable, and loosely coupled web services. Understanding these constraints is fundamental to designing true RESTful APIs.

## The 6 REST Constraints

### 1. Client-Server Architecture
- **Separation of concerns**: Client handles UI/user experience, server handles data storage and business logic
- **Independence**: Client and server can evolve independently
- **Scalability**: Server can serve multiple clients

```
Client (Browser, Mobile App) ←→ Server (API, Database)
```

### 2. Stateless
- **No session state**: Server doesn't store client context between requests
- **Self-contained requests**: Each request contains all information needed to process it
- **Scalability**: Easier to scale horizontally, any server can handle any request

```http
# Each request is independent and complete
GET /api/users/123
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
Accept: application/json
```

### 3. Cacheable
- **Response caching**: Responses should be explicitly marked as cacheable or non-cacheable
- **Performance**: Reduces client-server interactions
- **Network efficiency**: Reduces bandwidth usage

```http
HTTP/1.1 200 OK
Cache-Control: max-age=3600, public
ETag: "123456789"
Last-Modified: Wed, 21 Oct 2015 07:28:00 GMT
```

### 4. Uniform Interface
Four sub-constraints that define how client and server interact:

**a) Resource Identification (URIs)**
```http
GET /api/users/123        # Specific user
GET /api/users/123/posts  # User's posts
```

**b) Resource Manipulation through Representations**
```http
PUT /api/users/123
Content-Type: application/json

{
  "name": "John Doe",
  "email": "john@example.com"
}
```

**c) Self-Descriptive Messages**
```http
POST /api/users
Content-Type: application/json
Accept: application/json

{
  "name": "Jane Doe",
  "email": "jane@example.com"
}
```

**d) Hypermedia as Engine of Application State (HATEOAS)**
```json
{
  "id": 123,
  "name": "John Doe",
  "links": {
    "self": "/api/users/123",
    "posts": "/api/users/123/posts",
    "edit": "/api/users/123"
  }
}
```

### 5. Layered System
- **Hierarchical layers**: Client can't tell if connected directly to server or intermediary
- **Intermediaries**: Load balancers, proxies, gateways, caches
- **Encapsulation**: Each layer only knows about immediate layers

```
Client → CDN → Load Balancer → API Gateway → Microservice → Database
```

### 6. Code on Demand (Optional)
- **Executable code**: Server can send executable code to client
- **Dynamic functionality**: Extend client functionality without updating
- **Examples**: JavaScript, Java applets (rarely used in modern APIs)

## REST Maturity Model (Richardson Model)

### Level 0: HTTP as Transport
```http
POST /api/endpoint
<request>getUserData</request>
```

### Level 1: Resources
```http
GET /api/users/123
POST /api/users
```

### Level 2: HTTP Verbs
```http
GET /api/users/123      # Read
POST /api/users         # Create
PUT /api/users/123      # Update
DELETE /api/users/123   # Delete
```

### Level 3: Hypermedia Controls (HATEOAS)
```json
{
  "id": 123,
  "name": "John Doe",
  "_links": {
    "self": {"href": "/api/users/123"},
    "edit": {"href": "/api/users/123", "method": "PUT"},
    "delete": {"href": "/api/users/123", "method": "DELETE"}
  }
}
```

## Practical Examples

### RESTful User Management API

**Create User**
```http
POST /api/users
Content-Type: application/json

{
  "name": "John Doe",
  "email": "john@example.com",
  "role": "developer"
}

Response: 201 Created
Location: /api/users/123
{
  "id": 123,
  "name": "John Doe",
  "email": "john@example.com",
  "role": "developer",
  "created_at": "2025-08-27T10:30:00Z"
}
```

**Get User**
```http
GET /api/users/123
Accept: application/json

Response: 200 OK
{
  "id": 123,
  "name": "John Doe",
  "email": "john@example.com",
  "role": "developer",
  "last_login": "2025-08-27T09:15:00Z"
}
```

**Update User**
```http
PUT /api/users/123
Content-Type: application/json

{
  "name": "John Smith",
  "email": "john.smith@example.com",
  "role": "senior_developer"
}

Response: 200 OK
{
  "id": 123,
  "name": "John Smith",
  "email": "john.smith@example.com",
  "role": "senior_developer",
  "updated_at": "2025-08-27T10:45:00Z"
}
```

**Delete User**
```http
DELETE /api/users/123

Response: 204 No Content
```

### Collection Operations
```http
# Get all users with filtering
GET /api/users?role=developer&status=active&page=1&limit=20

# Search users
GET /api/users?search=john&sort=created_at&order=desc

Response: 200 OK
{
  "data": [
    {
      "id": 123,
      "name": "John Doe",
      "email": "john@example.com"
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 156,
    "pages": 8
  }
}
```

## Common Mistakes to Avoid

### 1. Using Verbs in URLs
```http
# Wrong
POST /api/createUser
GET /api/getUser/123
POST /api/updateUser/123

# Correct
POST /api/users
GET /api/users/123
PUT /api/users/123
```

### 2. Exposing Internal Structure
```http
# Wrong
GET /api/users/123/database_id/456

# Correct
GET /api/users/123
```

### 3. Not Using HTTP Status Codes Properly
```http
# Wrong
POST /api/users
{
  "status": "error",
  "message": "User already exists"
}
200 OK

# Correct
POST /api/users
{
  "error": "User already exists",
  "code": "USER_EXISTS"
}
409 Conflict
```

### 4. Breaking Stateless Constraint
```http
# Wrong - Server remembers session state
GET /api/users/next  # Depends on previous request

# Correct - Self-contained request
GET /api/users?page=2&limit=10
```

### 5. Inconsistent Resource Naming
```http
# Wrong
GET /api/user/123
GET /api/users-list
GET /api/getUserPosts/123

# Correct
GET /api/users/123
GET /api/users
GET /api/users/123/posts
```

## Interview Questions

1. **Q: What are the 6 REST constraints and why are they important?**
   **A:** Client-Server (separation of concerns), Stateless (scalability), Cacheable (performance), Uniform Interface (standardization), Layered System (flexibility), Code on Demand (optional extensibility). They ensure scalability, reliability, and maintainability.

2. **Q: Explain the difference between stateful and stateless communication.**
   **A:** Stateless means each request contains all information needed for processing, and the server doesn't store client context. Stateful keeps session information on the server. REST requires stateless for better scalability and reliability.

3. **Q: What is HATEOAS and why is it considered the highest level of REST maturity?**
   **A:** Hypermedia as the Engine of Application State. Responses include links to related actions/resources, making the API self-descriptive and discoverable. It's Level 3 in Richardson's maturity model because it provides the ultimate decoupling.

4. **Q: How does the Uniform Interface constraint improve API design?**
   **A:** It standardizes how resources are identified (URIs), manipulated (HTTP methods), and represented (media types). This creates predictable, consistent APIs that are easier to understand and use.

5. **Q: What makes an API truly RESTful vs just REST-like?**
   **A:** True REST follows all 6 constraints, especially HATEOAS. Many APIs are REST-like because they use HTTP methods and resource URIs but don't implement hypermedia controls.

6. **Q: How does the cacheable constraint improve API performance?**
   **A:** Allows responses to be stored and reused, reducing server load and network traffic. Proper cache headers (Cache-Control, ETag) enable efficient caching strategies.

7. **Q: Why is the layered system constraint important for modern APIs?**
   **A:** Enables architectural flexibility with load balancers, CDNs, API gateways, and security layers without affecting client implementation. Supports microservices and distributed systems.

8. **Q: What are the benefits of the client-server constraint?**
   **A:** Separation of concerns allows independent evolution, better scalability, and platform independence. UI can change without affecting business logic and vice versa.

9. **Q: How do you design URLs that follow REST principles?**
   **A:** Use nouns (not verbs), represent resource hierarchies, use consistent naming conventions, leverage HTTP methods for actions, and avoid exposing implementation details.

10. **Q: What's the difference between REST and RESTful?**
    **A:** REST is the architectural style with specific constraints. RESTful describes services that follow REST principles. An API can be RESTful to varying degrees based on constraint adherence.
