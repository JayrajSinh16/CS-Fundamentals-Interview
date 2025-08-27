# Authentication & Authorization

Authentication and authorization are critical security components of REST APIs. Authentication verifies the identity of the client making the request (who you are), while authorization determines what that authenticated client is allowed to do (what you can access). Understanding various authentication methods and authorization patterns is essential for building secure APIs.

## Authentication vs Authorization

### Authentication (AuthN)
**Purpose**: Verify the identity of the user or client
**Question**: "Who are you?"

```http
# Authentication example
POST /api/auth/login
Content-Type: application/json

{
  "username": "john@example.com",
  "password": "securepassword123"
}

Response: 200 OK
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "Bearer",
  "expires_in": 3600,
  "user": {
    "id": 123,
    "name": "John Doe",
    "email": "john@example.com"
  }
}
```

### Authorization (AuthZ)
**Purpose**: Determine what the authenticated user can access
**Question**: "What are you allowed to do?"

```http
# Authorization example
GET /api/admin/users
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

# If user doesn't have admin role
Response: 403 Forbidden
{
  "error": "insufficient_permissions",
  "message": "Admin role required to access user management",
  "required_role": "admin",
  "user_roles": ["user"]
}

# If user has admin role
Response: 200 OK
{
  "users": [...],
  "total": 150
}
```

## Authentication Methods

### 1. Bearer Token Authentication
Most common method for REST APIs, typically using JWT tokens.

```http
# Login to get token
POST /api/auth/login
Content-Type: application/json

{
  "email": "user@example.com",
  "password": "password123"
}

Response: 200 OK
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c",
  "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "Bearer",
  "expires_in": 3600
}

# Use token in subsequent requests
GET /api/profile
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

Response: 200 OK
{
  "id": 123,
  "name": "John Doe",
  "email": "user@example.com",
  "roles": ["user"]
}
```

### JWT Token Structure
```json
{
  "header": {
    "alg": "HS256",
    "typ": "JWT"
  },
  "payload": {
    "sub": "123",
    "name": "John Doe",
    "email": "user@example.com",
    "roles": ["user", "editor"],
    "permissions": ["read:profile", "write:posts"],
    "iat": 1693123200,
    "exp": 1693126800,
    "iss": "https://api.example.com",
    "aud": "web-app"
  },
  "signature": "..."
}
```

### 2. API Key Authentication
Simple authentication method using pre-shared keys.

```http
# API Key in header (recommended)
GET /api/data
X-API-Key: sk_live_1234567890abcdef
Authorization: Bearer user_token_if_needed

# API Key in query parameter (less secure)
GET /api/public-data?api_key=pk_test_1234567890abcdef

# API Key in Authorization header
GET /api/data
Authorization: ApiKey sk_live_1234567890abcdef

# Response for valid API key
Response: 200 OK
X-RateLimit-Limit: 10000
X-RateLimit-Remaining: 9999
{
  "data": [...]
}

# Response for invalid API key
Response: 401 Unauthorized
{
  "error": "invalid_api_key",
  "message": "The provided API key is invalid"
}
```

### 3. Basic Authentication
Simple username/password authentication (should only be used over HTTPS).

```http
# Basic Authentication
GET /api/protected
Authorization: Basic dXNlcm5hbWU6cGFzc3dvcmQ=
# Base64 encoded "username:password"

# Response for valid credentials
Response: 200 OK
{
  "message": "Access granted",
  "user": "username"
}

# Response for invalid credentials
Response: 401 Unauthorized
WWW-Authenticate: Basic realm="API"
{
  "error": "invalid_credentials",
  "message": "Invalid username or password"
}
```

### 4. OAuth 2.0
Industry-standard authorization framework for secure API access.

```http
# Authorization Code Flow

# Step 1: Redirect user to authorization server
GET https://auth.example.com/oauth/authorize?
    response_type=code&
    client_id=your_client_id&
    redirect_uri=https://yourapp.com/callback&
    scope=read:profile write:posts&
    state=random_state_string

# Step 2: User authorizes, server redirects back
# https://yourapp.com/callback?code=auth_code_123&state=random_state_string

# Step 3: Exchange code for tokens
POST https://auth.example.com/oauth/token
Content-Type: application/x-www-form-urlencoded

grant_type=authorization_code&
code=auth_code_123&
client_id=your_client_id&
client_secret=your_client_secret&
redirect_uri=https://yourapp.com/callback

Response: 200 OK
{
  "access_token": "2YotnFZFEjr1zCsicMWpAA",
  "token_type": "Bearer",
  "expires_in": 3600,
  "refresh_token": "tGzv3JOkF0XG5Qx2TlKWIA",
  "scope": "read:profile write:posts"
}

# Step 4: Use access token for API calls
GET /api/profile
Authorization: Bearer 2YotnFZFEjr1zCsicMWpAA
```

### 5. Client Credentials Flow (Machine-to-Machine)
```http
# Service-to-service authentication
POST https://auth.example.com/oauth/token
Content-Type: application/x-www-form-urlencoded

grant_type=client_credentials&
client_id=service_client_id&
client_secret=service_client_secret&
scope=api:read api:write

Response: 200 OK
{
  "access_token": "2YotnFZFEjr1zCsicMWpAA",
  "token_type": "Bearer",
  "expires_in": 3600,
  "scope": "api:read api:write"
}

# Use token for service calls
GET /api/internal/data
Authorization: Bearer 2YotnFZFEjr1zCsicMWpAA
```

## Authorization Patterns

### 1. Role-Based Access Control (RBAC)
Users are assigned roles, and roles have permissions.

```http
# User with editor role
GET /api/posts/123
Authorization: Bearer token_with_editor_role

Response: 200 OK
{
  "id": 123,
  "title": "My Post",
  "content": "...",
  "actions": ["edit", "delete", "publish"]
}

# User with viewer role
GET /api/posts/123
Authorization: Bearer token_with_viewer_role

Response: 200 OK
{
  "id": 123,
  "title": "My Post",
  "content": "...",
  "actions": ["view"]
}

# Role hierarchy example
{
  "roles": {
    "admin": {
      "permissions": ["*"]
    },
    "editor": {
      "permissions": ["read:*", "write:posts", "write:pages", "delete:own_posts"]
    },
    "author": {
      "permissions": ["read:*", "write:own_posts", "delete:own_posts"]
    },
    "viewer": {
      "permissions": ["read:published"]
    }
  }
}
```

### 2. Attribute-Based Access Control (ABAC)
Access decisions based on attributes of user, resource, and environment.

```http
# ABAC policy evaluation
GET /api/documents/123
Authorization: Bearer token

# Server evaluates policy:
# - User attributes: department=finance, level=senior
# - Resource attributes: classification=confidential, owner=finance
# - Environment: time=business_hours, location=office_network
# - Action: read

# Policy: Allow if (user.department == resource.owner) 
#         AND (user.level >= 'senior' OR resource.classification != 'confidential')
#         AND environment.time == 'business_hours'

Response: 200 OK
{
  "id": 123,
  "title": "Financial Report Q3",
  "content": "...",
  "access_granted_because": "Department match and senior level"
}
```

### 3. Resource-Based Permissions
Permissions tied to specific resources.

```http
# Check specific resource permissions
GET /api/projects/456/permissions
Authorization: Bearer user_token

Response: 200 OK
{
  "project_id": 456,
  "user_permissions": [
    "read",
    "write",
    "delete",
    "manage_members"
  ],
  "inherited_from": [
    {
      "source": "team_membership",
      "team": "development",
      "permissions": ["read", "write"]
    },
    {
      "source": "direct_assignment",
      "permissions": ["delete", "manage_members"]
    }
  ]
}

# Access resource with specific permission
POST /api/projects/456/members
Authorization: Bearer user_token
Content-Type: application/json

{
  "user_id": 789,
  "role": "contributor"
}

# Success if user has 'manage_members' permission
Response: 201 Created
```

### 4. Scope-Based Authorization
OAuth-style scopes for fine-grained permissions.

```http
# Token with specific scopes
{
  "access_token": "...",
  "scopes": [
    "read:profile",
    "write:posts", 
    "read:posts",
    "delete:own_posts",
    "admin:users"
  ]
}

# API enforces scopes
GET /api/profile
Authorization: Bearer token_with_read_profile_scope
# Requires: read:profile scope

POST /api/posts
Authorization: Bearer token_with_write_posts_scope
# Requires: write:posts scope

DELETE /api/users/123
Authorization: Bearer token_without_admin_scope
Response: 403 Forbidden
{
  "error": "insufficient_scope",
  "message": "This operation requires 'admin:users' scope",
  "required_scopes": ["admin:users"],
  "available_scopes": ["read:profile", "write:posts", "read:posts", "delete:own_posts"]
}
```

## Token Management

### Token Refresh Flow
```http
# Access token expires
GET /api/protected
Authorization: Bearer expired_access_token

Response: 401 Unauthorized
{
  "error": "token_expired",
  "message": "Access token has expired"
}

# Use refresh token to get new access token
POST /api/auth/refresh
Content-Type: application/json

{
  "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}

Response: 200 OK
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "Bearer",
  "expires_in": 3600,
  "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}

# Retry original request with new token
GET /api/protected
Authorization: Bearer new_access_token

Response: 200 OK
```

### Token Revocation
```http
# Revoke access token
POST /api/auth/revoke
Content-Type: application/json
Authorization: Bearer token_to_revoke

{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type_hint": "access_token"
}

Response: 200 OK
{
  "message": "Token revoked successfully"
}

# Revoke all user sessions
POST /api/auth/revoke-all
Authorization: Bearer current_token

Response: 200 OK
{
  "message": "All tokens revoked",
  "revoked_sessions": 5
}
```

### Token Introspection
```http
# Check token validity and get metadata
POST /api/auth/introspect
Content-Type: application/x-www-form-urlencoded
Authorization: Basic Y2xpZW50X2lkOmNsaWVudF9zZWNyZXQ=

token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

Response: 200 OK
{
  "active": true,
  "client_id": "web_app",
  "username": "john@example.com",
  "scope": "read:profile write:posts",
  "exp": 1693126800,
  "iat": 1693123200,
  "sub": "123",
  "aud": "api.example.com"
}
```

## Security Best Practices

### 1. Secure Token Storage
```javascript
// Client-side token storage options

// Most secure: Memory only (lost on page refresh)
class TokenManager {
  constructor() {
    this.accessToken = null;
  }
  
  setToken(token) {
    this.accessToken = token;
  }
  
  getToken() {
    return this.accessToken;
  }
}

// Moderate security: sessionStorage (lost on tab close)
sessionStorage.setItem('access_token', token);
const token = sessionStorage.getItem('access_token');

// Less secure: localStorage (persists until manually cleared)
localStorage.setItem('access_token', token);

// Most secure for refresh tokens: httpOnly cookies
// Set by server, not accessible to JavaScript
Set-Cookie: refresh_token=abc123; HttpOnly; Secure; SameSite=Strict; Max-Age=604800
```

### 2. Token Validation
```http
# Server validates tokens on each request
GET /api/protected
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

# Server checks:
# 1. Token signature validity
# 2. Token expiration
# 3. Token not in revocation list
# 4. Required scopes/permissions
# 5. Issuer and audience claims

# Invalid token response
Response: 401 Unauthorized
WWW-Authenticate: Bearer error="invalid_token", error_description="Token signature is invalid"

# Expired token response
Response: 401 Unauthorized
WWW-Authenticate: Bearer error="invalid_token", error_description="Token has expired"

# Insufficient permissions
Response: 403 Forbidden
{
  "error": "insufficient_permissions",
  "required_permissions": ["admin:users"],
  "user_permissions": ["user:profile"]
}
```

### 3. Rate Limiting by Authentication
```http
# Different rate limits for different auth levels
GET /api/data
X-API-Key: public_key

Response: 200 OK
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 99
X-RateLimit-Window: 3600

GET /api/data
Authorization: Bearer premium_user_token

Response: 200 OK
X-RateLimit-Limit: 10000
X-RateLimit-Remaining: 9999
X-RateLimit-Window: 3600

# Rate limit exceeded
Response: 429 Too Many Requests
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 0
X-RateLimit-Reset: 1693126800
Retry-After: 3600
```

## Multi-Factor Authentication (MFA)

### MFA Challenge Flow
```http
# Initial login
POST /api/auth/login
Content-Type: application/json

{
  "email": "user@example.com",
  "password": "password123"
}

Response: 202 Accepted
{
  "challenge_required": true,
  "challenge_type": "totp",
  "challenge_token": "temp_token_abc123",
  "message": "Please provide your TOTP code"
}

# Submit MFA challenge
POST /api/auth/challenge
Content-Type: application/json

{
  "challenge_token": "temp_token_abc123",
  "challenge_type": "totp",
  "code": "123456"
}

Response: 200 OK
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "Bearer",
  "expires_in": 3600,
  "mfa_verified": true
}
```

### Backup Codes
```http
# Generate backup codes
POST /api/auth/mfa/backup-codes
Authorization: Bearer user_token

Response: 200 OK
{
  "backup_codes": [
    "12345678",
    "87654321",
    "11111111",
    "22222222"
  ],
  "message": "Store these codes securely. Each can only be used once."
}

# Use backup code
POST /api/auth/challenge
Content-Type: application/json

{
  "challenge_token": "temp_token_abc123",
  "challenge_type": "backup_code",
  "code": "12345678"
}

Response: 200 OK
{
  "access_token": "...",
  "backup_code_used": "12345678",
  "remaining_backup_codes": 3
}
```

## Error Handling

### Authentication Errors
```http
# Missing authentication
GET /api/protected

Response: 401 Unauthorized
WWW-Authenticate: Bearer realm="api"
{
  "error": "authentication_required",
  "message": "This endpoint requires authentication",
  "auth_url": "/api/auth/login"
}

# Invalid credentials
POST /api/auth/login
{
  "email": "user@example.com",
  "password": "wrong_password"
}

Response: 401 Unauthorized
{
  "error": "invalid_credentials",
  "message": "Invalid email or password",
  "login_attempts_remaining": 2
}

# Account locked
POST /api/auth/login
{
  "email": "locked@example.com",
  "password": "password123"
}

Response: 423 Locked
{
  "error": "account_locked",
  "message": "Account locked due to too many failed login attempts",
  "unlock_time": "2025-08-27T12:00:00Z",
  "contact_support": "support@example.com"
}
```

### Authorization Errors
```http
# Insufficient permissions
DELETE /api/admin/users/123
Authorization: Bearer regular_user_token

Response: 403 Forbidden
{
  "error": "insufficient_permissions",
  "message": "Admin role required for this operation",
  "required_role": "admin",
  "user_roles": ["user"],
  "contact_admin": "admin@example.com"
}

# Resource not owned by user
DELETE /api/posts/456
Authorization: Bearer user_token

Response: 403 Forbidden
{
  "error": "resource_not_owned",
  "message": "You can only delete your own posts",
  "resource_owner": "other_user_id",
  "your_id": "current_user_id"
}
```

## Interview Questions

1. **Q: What's the difference between authentication and authorization?**
   **A:** Authentication verifies identity (who you are) - like checking ID at a door. Authorization determines permissions (what you can do) - like checking if your ticket allows VIP access. Authentication comes first, then authorization.

2. **Q: Why is Bearer token authentication preferred over Basic authentication?**
   **A:** Bearer tokens are stateless, can contain user info/permissions, have expiration times, can be revoked, and don't require sending passwords with every request. Basic auth sends credentials every time, which is less secure.

3. **Q: How do JWT tokens work and what are their advantages?**
   **A:** JWTs are self-contained tokens with header, payload, and signature. Advantages: stateless (no server storage), contain user info, cryptographically signed, can be verified without database lookup, and support expiration/scopes.

4. **Q: What's the difference between access tokens and refresh tokens?**
   **A:** Access tokens are short-lived (minutes/hours) and used for API calls. Refresh tokens are long-lived (days/weeks) and used only to get new access tokens when they expire. This limits exposure if access tokens are compromised.

5. **Q: How do you implement role-based access control in APIs?**
   **A:** Define roles with specific permissions, assign roles to users, include role/permission info in tokens, check permissions on each API endpoint, and return 403 Forbidden for insufficient permissions.

6. **Q: What are OAuth 2.0 scopes and how are they used?**
   **A:** Scopes define granular permissions (read:profile, write:posts). They limit what an access token can do, enable principle of least privilege, and allow users to grant specific permissions to third-party apps.

7. **Q: How should you handle token expiration in client applications?**
   **A:** Detect 401 responses, automatically attempt token refresh using refresh token, retry original request with new token, redirect to login if refresh fails, and handle this transparently for users.

8. **Q: What security considerations apply to API keys?**
   **A:** Use HTTPS only, include in headers (not URLs), implement rate limiting, support key rotation, use different keys for different environments, monitor usage, and provide key management interfaces.

9. **Q: How do you implement proper session management in stateless APIs?**
   **A:** Use JWTs for stateless sessions, implement token refresh mechanisms, support token revocation lists, use secure token storage, implement proper logout (token invalidation), and monitor for suspicious activity.

10. **Q: What's the best way to handle authorization for hierarchical resources?**
    **A:** Use resource-based permissions, implement permission inheritance (team > project > task), check permissions at each level, cache permission results, use efficient permission evaluation algorithms, and provide clear permission APIs.
