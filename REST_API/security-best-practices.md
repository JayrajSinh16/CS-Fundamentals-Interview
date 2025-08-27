# Security Best Practices for REST APIs

Security is paramount in REST API design as APIs often expose sensitive data and critical business operations. Implementing comprehensive security measures protects against various attack vectors and ensures data confidentiality, integrity, and availability. This guide covers essential security practices for building secure REST APIs.

## Security Principles

### 1. Defense in Depth
- **Multiple Security Layers**: Implement security at network, application, and data levels
- **Fail-Safe Defaults**: Default to secure configurations and explicit permissions
- **Least Privilege**: Grant minimum necessary permissions for API access
- **Zero Trust**: Verify every request regardless of source

### 2. Security by Design
- **Threat Modeling**: Identify potential attack vectors during API design
- **Security Requirements**: Define security requirements alongside functional requirements
- **Regular Reviews**: Conduct security reviews throughout development lifecycle
- **Penetration Testing**: Regular security testing and vulnerability assessments

## Authentication Mechanisms

### 1. Bearer Token Authentication

Most common authentication method using tokens in Authorization header.

```http
# Login request to obtain token
POST /api/auth/login
Content-Type: application/json
{
  "username": "john.doe@example.com",
  "password": "SecurePassword123!",
  "remember_me": false
}

Response: 200 OK
Set-Cookie: refresh_token=abc123...; HttpOnly; Secure; SameSite=Strict; Max-Age=2592000
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "Bearer",
  "expires_in": 3600,
  "refresh_token": "def456...",
  "scope": "read write",
  "user_info": {
    "id": 123,
    "username": "john.doe@example.com",
    "roles": ["user", "premium"]
  }
}

# Authenticated API request
GET /api/users/profile
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
Response: 200 OK
{
  "id": 123,
  "username": "john.doe@example.com",
  "profile": {...}
}

# Token refresh
POST /api/auth/refresh
Authorization: Bearer def456...
# OR with cookie
Cookie: refresh_token=abc123...

Response: 200 OK
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "expires_in": 3600
}
```

### 2. API Key Authentication

Simple authentication using API keys for service-to-service communication.

```http
# API key in header (preferred)
GET /api/data/analytics
X-API-Key: ak_live_1234567890abcdef
# OR
Authorization: ApiKey ak_live_1234567890abcdef

Response: 200 OK
X-RateLimit-Remaining: 99
{
  "data": [...],
  "api_key_info": {
    "key_id": "ak_live_1234567890abcdef",
    "permissions": ["read_analytics", "read_reports"],
    "rate_limit": 100,
    "quota_remaining": 850
  }
}

# API key in query parameter (less secure)
GET /api/public/weather?api_key=ak_live_1234567890abcdef
Response: 200 OK
{
  "weather_data": {...},
  "warning": "API key in URL is less secure. Use headers instead."
}

# Invalid API key
GET /api/data/analytics
X-API-Key: invalid_key
Response: 401 Unauthorized
{
  "error": "invalid_api_key",
  "message": "The provided API key is invalid or expired",
  "code": "AUTH_001"
}
```

### 3. OAuth 2.0 Implementation

Industry-standard authorization framework for secure API access.

```http
# Authorization Code Flow - Step 1: Redirect to authorization server
GET https://auth.example.com/oauth/authorize?
    response_type=code&
    client_id=your_client_id&
    redirect_uri=https://yourapp.com/callback&
    scope=read_profile write_posts&
    state=random_state_string

# Step 2: User grants permission, redirects back
GET https://yourapp.com/callback?
    code=authorization_code_here&
    state=random_state_string

# Step 3: Exchange authorization code for tokens
POST https://auth.example.com/oauth/token
Content-Type: application/x-www-form-urlencoded
Authorization: Basic base64(client_id:client_secret)

grant_type=authorization_code&
code=authorization_code_here&
redirect_uri=https://yourapp.com/callback

Response: 200 OK
{
  "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "Bearer", 
  "expires_in": 3600,
  "refresh_token": "refresh_token_here",
  "scope": "read_profile write_posts"
}

# Step 4: Use access token for API calls
GET /api/users/me
Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...

Response: 200 OK
{
  "user_id": 12345,
  "username": "john_doe",
  "scopes": ["read_profile", "write_posts"]
}

# Client Credentials Flow (for service-to-service)
POST https://auth.example.com/oauth/token
Content-Type: application/x-www-form-urlencoded
Authorization: Basic base64(client_id:client_secret)

grant_type=client_credentials&
scope=api_access

Response: 200 OK
{
  "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "Bearer",
  "expires_in": 7200,
  "scope": "api_access"
}
```

## Authorization and Access Control

### 1. Role-Based Access Control (RBAC)

Define roles and permissions for different types of users.

```http
# Check user permissions
GET /api/admin/users
Authorization: Bearer user_token

Response: 403 Forbidden
{
  "error": "insufficient_permissions",
  "message": "Admin role required for this operation",
  "required_roles": ["admin"],
  "user_roles": ["user", "premium"],
  "code": "AUTHZ_001"
}

# Admin user access
GET /api/admin/users
Authorization: Bearer admin_token

Response: 200 OK
X-User-Roles: admin,super_admin
{
  "users": [
    {
      "id": 123,
      "username": "john.doe",
      "roles": ["user", "premium"],
      "status": "active"
    }
  ],
  "permissions_used": ["read_all_users"]
}

# Resource-specific permissions
GET /api/documents/456
Authorization: Bearer user_token

Response: 200 OK
{
  "id": 456,
  "title": "Confidential Document",
  "content": "...",
  "permissions": {
    "read": true,
    "write": false,
    "delete": false,
    "share": true
  },
  "access_reason": "document_owner"
}

# Permission denied for specific resource
DELETE /api/documents/456
Authorization: Bearer user_token

Response: 403 Forbidden
{
  "error": "insufficient_permissions",
  "message": "Delete permission required for this document",
  "resource_id": 456,
  "required_permission": "delete_document",
  "current_permissions": ["read_document", "share_document"]
}
```

### 2. Attribute-Based Access Control (ABAC)

Fine-grained access control based on attributes.

```http
# Request with context attributes
GET /api/medical-records/patient/789
Authorization: Bearer doctor_token
X-Request-Context: department=cardiology,shift=day,location=hospital_main

Response: 200 OK
{
  "patient_id": 789,
  "records": [...],
  "access_decision": {
    "granted": true,
    "reasons": [
      "User is licensed doctor",
      "Patient assigned to user's department", 
      "Access during authorized hours",
      "Request from authorized location"
    ],
    "conditions": [
      "Audit log entry created",
      "Patient notification sent"
    ]
  }
}

# Access denied due to context
GET /api/medical-records/patient/789
Authorization: Bearer doctor_token  
X-Request-Context: department=cardiology,shift=night,location=home

Response: 403 Forbidden
{
  "error": "access_denied",
  "message": "Medical records access not allowed from home location",
  "policy_violations": [
    {
      "rule": "medical_records_location_policy",
      "requirement": "Access only from hospital premises",
      "actual": "home"
    }
  ],
  "appeal_process": "Contact administrator for emergency access"
}
```

## Input Validation and Sanitization

### 1. Request Validation

Validate all input data to prevent injection attacks and ensure data integrity.

```http
# Valid request with proper validation
POST /api/users
Content-Type: application/json
{
  "username": "john_doe123",
  "email": "john@example.com",
  "age": 30,
  "profile": {
    "bio": "Software developer passionate about APIs"
  }
}

Response: 201 Created
{
  "id": 123,
  "username": "john_doe123",
  "email": "john@example.com",
  "validation_passed": true
}

# Invalid request with validation errors
POST /api/users
Content-Type: application/json
{
  "username": "jd",
  "email": "invalid-email",
  "age": -5,
  "profile": {
    "bio": "<script>alert('xss')</script>"
  }
}

Response: 400 Bad Request
{
  "error": "validation_failed",
  "message": "Request validation failed",
  "validation_errors": [
    {
      "field": "username",
      "error": "must be at least 3 characters long",
      "value": "jd",
      "constraint": "min_length:3"
    },
    {
      "field": "email",
      "error": "must be a valid email address",
      "value": "invalid-email",
      "constraint": "email_format"
    },
    {
      "field": "age",
      "error": "must be a positive integer",
      "value": -5,
      "constraint": "min:0"
    },
    {
      "field": "profile.bio",
      "error": "contains potentially dangerous content",
      "sanitized_value": "alert('xss')",
      "constraint": "no_script_tags"
    }
  ],
  "code": "VALIDATION_001"
}

# SQL injection attempt
GET /api/users?search='; DROP TABLE users; --
Response: 400 Bad Request
{
  "error": "invalid_input",
  "message": "Potentially dangerous input detected",
  "security_violation": {
    "type": "sql_injection_attempt",
    "field": "search",
    "pattern_matched": "sql_keywords_with_special_chars"
  },
  "incident_id": "SEC_20250827_001"
}
```

### 2. Output Encoding

Properly encode output to prevent XSS and other injection attacks.

```http
# Safe output encoding
GET /api/posts/123
Response: 200 OK
Content-Type: application/json
{
  "id": 123,
  "title": "My Blog Post",
  "content": "Check out this &lt;script&gt; tag!",  // HTML encoded
  "author": {
    "name": "John &amp; Jane Doe",              // Ampersand encoded
    "bio": "We love \"quotes\" and 'apostrophes'"  // Properly escaped
  },
  "comments": [
    {
      "text": "Great post! Visit my site: http://example.com",
      "sanitized": true,
      "original_length": 45,
      "cleaned_length": 41
    }
  ]
}

# Content Security Policy headers
GET /api/admin/dashboard
Response: 200 OK
Content-Security-Policy: default-src 'self'; script-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline'
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
X-XSS-Protection: 1; mode=block
{
  "dashboard_data": {...}
}
```

## Data Protection

### 1. Encryption in Transit

Ensure all API communications are encrypted using HTTPS/TLS.

```http
# Secure HTTPS request
GET https://api.example.com/sensitive-data
Authorization: Bearer token
Response: 200 OK
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
{
  "sensitive_data": "encrypted_payload_here",
  "encryption_info": {
    "tls_version": "1.3",
    "cipher_suite": "TLS_AES_256_GCM_SHA384",
    "certificate_valid": true
  }
}

# Redirect insecure HTTP to HTTPS
GET http://api.example.com/data
Response: 301 Moved Permanently
Location: https://api.example.com/data
Strict-Transport-Security: max-age=31536000

# Certificate pinning validation
GET https://api.example.com/data
# Client validates certificate against pinned certificate
Response: 200 OK
Public-Key-Pins: pin-sha256="base64+primary+key"; pin-sha256="base64+backup+key"; max-age=5184000
```

### 2. Encryption at Rest

Protect stored data using encryption.

```http
# Request for encrypted data
GET /api/users/123/sensitive-info
Authorization: Bearer admin_token

Response: 200 OK
{
  "user_id": 123,
  "ssn": "***-**-1234",                    // Masked sensitive data
  "credit_card": "**** **** **** 5678",   // Partially masked
  "encrypted_fields": [
    "ssn", "credit_card", "medical_history"
  ],
  "encryption_status": {
    "database_encrypted": true,
    "encryption_algorithm": "AES-256-GCM",
    "key_management": "HSM"
  }
}

# Data classification headers
GET /api/documents/financial-report
Authorization: Bearer analyst_token

Response: 200 OK
X-Data-Classification: CONFIDENTIAL
X-Encryption-Status: encrypted
X-Access-Controls: role_based,location_restricted
{
  "document_id": 456,
  "classification": "CONFIDENTIAL",
  "content": "encrypted_base64_content_here",
  "access_log": {
    "accessed_by": "analyst_user",
    "access_time": "2025-08-27T10:30:00Z",
    "purpose": "financial_analysis"
  }
}
```

## Rate Limiting and DoS Protection

### 1. Request Rate Limiting

Implement rate limiting to prevent abuse and ensure fair usage.

```http
# Normal request within limits
GET /api/data
X-API-Key: client_key_123

Response: 200 OK
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 995
X-RateLimit-Reset: 1693123200
X-RateLimit-Window: 3600
{
  "data": [...]
}

# Rate limit exceeded
GET /api/data
X-API-Key: client_key_123

Response: 429 Too Many Requests
Retry-After: 300
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 0
X-RateLimit-Reset: 1693123200
{
  "error": "rate_limit_exceeded",
  "message": "Request rate limit exceeded",
  "limit": 1000,
  "window": "1 hour",
  "retry_after": 300,
  "upgrade_options": {
    "premium_plan": {
      "limit": 10000,
      "cost": "$99/month"
    }
  }
}

# Different limits for different endpoints
POST /api/auth/login
Response: 200 OK
X-RateLimit-Limit-Auth: 5
X-RateLimit-Remaining-Auth: 4
X-RateLimit-Reset-Auth: 1693119600
X-RateLimit-Window-Auth: 900
{
  "access_token": "..."
}

# Burst protection
GET /api/data
X-API-Key: client_key_123
# 50 requests in 10 seconds

Response: 429 Too Many Requests
X-RateLimit-Burst-Limit: 10
X-RateLimit-Burst-Remaining: 0
X-RateLimit-Burst-Reset: 1693119610
{
  "error": "burst_limit_exceeded",
  "message": "Too many requests in short time period",
  "burst_limit": 10,
  "burst_window": "10 seconds"
}
```

### 2. DDoS Protection

Implement distributed denial of service attack protection.

```http
# Request during DDoS attack
GET /api/data
Response: 503 Service Unavailable
Retry-After: 60
X-DDoS-Protection: active
{
  "error": "service_temporarily_unavailable",
  "message": "Service is under heavy load, please retry later",
  "estimated_recovery": "2025-08-27T10:35:00Z",
  "status_page": "https://status.example.com"
}

# Challenge-response for suspicious traffic
GET /api/data
Response: 202 Accepted
X-Challenge-Required: true
{
  "challenge_type": "captcha",
  "challenge_token": "challenge_token_here",
  "instructions": "Complete CAPTCHA verification",
  "challenge_url": "https://api.example.com/challenge/captcha"
}

# After successful challenge
GET /api/data
X-Challenge-Token: verified_challenge_token
Response: 200 OK
X-DDoS-Protection: challenge_passed
{
  "data": [...]
}
```

## Security Headers

### 1. Essential Security Headers

Implement security headers to protect against common web vulnerabilities.

```http
GET /api/data
Response: 200 OK
# Security headers
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
Content-Security-Policy: default-src 'self'; script-src 'self'; object-src 'none'
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
X-XSS-Protection: 1; mode=block
Referrer-Policy: strict-origin-when-cross-origin
Permissions-Policy: geolocation=(), microphone=(), camera=()
Cross-Origin-Embedder-Policy: require-corp
Cross-Origin-Opener-Policy: same-origin
Cross-Origin-Resource-Policy: same-origin

# CORS headers for cross-origin requests
Access-Control-Allow-Origin: https://trusted-app.com
Access-Control-Allow-Methods: GET, POST, PUT, DELETE, OPTIONS
Access-Control-Allow-Headers: Authorization, Content-Type, X-Requested-With
Access-Control-Allow-Credentials: true
Access-Control-Max-Age: 86400

{
  "data": [...]
}

# API-specific security headers
GET /api/sensitive-operation
Authorization: Bearer token

Response: 200 OK
Cache-Control: no-store, no-cache, must-revalidate, private
Pragma: no-cache
Expires: 0
X-Robots-Tag: noindex, nofollow, nosnippet, noarchive
{
  "sensitive_data": "..."
}
```

### 2. CORS Configuration

Properly configure Cross-Origin Resource Sharing for web applications.

```http
# Preflight request
OPTIONS /api/users
Origin: https://webapp.example.com
Access-Control-Request-Method: POST
Access-Control-Request-Headers: Authorization, Content-Type

Response: 204 No Content
Access-Control-Allow-Origin: https://webapp.example.com
Access-Control-Allow-Methods: GET, POST, PUT, DELETE, OPTIONS
Access-Control-Allow-Headers: Authorization, Content-Type, X-Request-ID
Access-Control-Allow-Credentials: true
Access-Control-Max-Age: 86400

# Actual CORS request
POST /api/users
Origin: https://webapp.example.com
Authorization: Bearer token
Content-Type: application/json

Response: 201 Created
Access-Control-Allow-Origin: https://webapp.example.com
Access-Control-Allow-Credentials: true
{
  "id": 123,
  "username": "newuser"
}

# CORS error for unauthorized origin
POST /api/users
Origin: https://malicious-site.com
Authorization: Bearer token

Response: 403 Forbidden
{
  "error": "cors_not_allowed",
  "message": "Origin not allowed by CORS policy",
  "allowed_origins": ["https://webapp.example.com", "https://mobile.example.com"]
}
```

## Security Monitoring and Logging

### 1. Security Event Logging

Log security-relevant events for monitoring and forensics.

```http
# Successful authentication
POST /api/auth/login
{
  "username": "john.doe@example.com",
  "password": "password"
}

Response: 200 OK
X-Request-ID: req_12345
{
  "access_token": "...",
  "security_event": {
    "event_type": "authentication_success",
    "user_id": 123,
    "ip_address": "192.168.1.100",
    "user_agent": "Mozilla/5.0...",
    "timestamp": "2025-08-27T10:30:00Z",
    "event_id": "sec_evt_67890"
  }
}

# Failed authentication attempt
POST /api/auth/login
{
  "username": "john.doe@example.com", 
  "password": "wrongpassword"
}

Response: 401 Unauthorized
X-Request-ID: req_12346
{
  "error": "authentication_failed",
  "security_event": {
    "event_type": "authentication_failure",
    "username": "john.doe@example.com",
    "ip_address": "192.168.1.100",
    "failure_reason": "invalid_password",
    "attempt_count": 3,
    "lockout_time": "2025-08-27T10:45:00Z"
  }
}

# Suspicious activity detection
GET /api/admin/sensitive-data
Authorization: Bearer suspicious_token
X-Forwarded-For: 10.0.0.1, 192.168.1.1, 203.0.113.195

Response: 403 Forbidden
X-Security-Alert: true
{
  "error": "suspicious_activity_detected",
  "security_alert": {
    "alert_type": "anomalous_access_pattern",
    "risk_score": 85,
    "indicators": [
      "Multiple IP addresses in request chain",
      "Access from new geographic location",
      "Unusual time of access",
      "Rapid sequential requests"
    ],
    "action_taken": "access_denied",
    "incident_id": "INC_20250827_001"
  }
}
```

### 2. Real-time Security Monitoring

Monitor API usage for security threats in real-time.

```http
# Security status endpoint
GET /api/security/status
Authorization: Bearer admin_token

Response: 200 OK
{
  "security_status": "elevated",
  "active_threats": [
    {
      "threat_type": "brute_force_attack",
      "target": "/api/auth/login",
      "source_ips": ["203.0.113.195", "203.0.113.196"],
      "attempts": 1500,
      "status": "mitigated"
    },
    {
      "threat_type": "rate_limit_abuse",
      "target": "/api/data/export",
      "source": "api_key_xyz789",
      "requests_per_minute": 500,
      "status": "monitoring"
    }
  ],
  "security_metrics": {
    "blocked_requests_24h": 2500,
    "authentication_failures_1h": 45,
    "rate_limit_violations_1h": 12,
    "suspicious_patterns_detected": 3
  }
}

# Automatic security response
GET /api/data
X-API-Key: compromised_key_123

Response: 403 Forbidden
X-Security-Action: automatic_block
{
  "error": "security_violation",
  "message": "API key has been automatically suspended due to suspicious activity",
  "violation_details": {
    "trigger": "anomalous_usage_pattern",
    "detection_time": "2025-08-27T10:30:00Z",
    "automated_response": "key_suspension",
    "contact": "security@example.com"
  },
  "remediation": {
    "steps": [
      "Contact security team for investigation",
      "Provide legitimate use case documentation",
      "Request new API key if confirmed compromised"
    ]
  }
}
```

## Interview Questions

1. **Q: What are the essential security headers for REST APIs and their purposes?**
   **A:** Strict-Transport-Security (HTTPS enforcement), Content-Security-Policy (XSS prevention), X-Content-Type-Options (MIME sniffing prevention), X-Frame-Options (clickjacking protection), CORS headers (cross-origin control), and Cache-Control (sensitive data caching prevention).

2. **Q: How do you implement proper authentication vs authorization in REST APIs?**
   **A:** Authentication verifies identity (who you are) using methods like JWT tokens, API keys, or OAuth. Authorization determines permissions (what you can do) using RBAC, ABAC, or resource-specific permissions. Always authenticate first, then authorize based on user roles and context.

3. **Q: What's the difference between symmetric and asymmetric encryption in API security?**
   **A:** Symmetric encryption uses the same key for encryption/decryption (faster, for data at rest). Asymmetric uses public/private key pairs (slower, for key exchange and digital signatures). APIs typically use asymmetric for initial handshake, then symmetric for data transmission.

4. **Q: How do you prevent and detect API abuse and DDoS attacks?**
   **A:** Implement rate limiting with different tiers, use IP whitelisting/blacklisting, deploy DDoS protection services, implement CAPTCHA challenges, monitor traffic patterns, set up automatic blocking rules, and use geographic restrictions where appropriate.

5. **Q: What are the security considerations for API versioning?**
   **A:** Maintain security patches across all supported versions, deprecate insecure versions promptly, implement version-specific security policies, audit security vulnerabilities across versions, and ensure new versions don't inherit old security flaws.

6. **Q: How do you secure API keys and tokens?**
   **A:** Store securely (never in code/URLs), use HTTPS only, implement token rotation, set appropriate expiration times, use least privilege principles, monitor usage patterns, implement token revocation, and consider using short-lived tokens with refresh mechanisms.

7. **Q: What's the OWASP API Security Top 10 and how do you address these risks?**
   **A:** Broken Object Level Authorization, Broken User Authentication, Excessive Data Exposure, Lack of Resources & Rate Limiting, Broken Function Level Authorization, Mass Assignment, Security Misconfiguration, Injection, Improper Assets Management, Insufficient Logging & Monitoring. Address through proper access controls, input validation, security headers, monitoring, and regular security testing.

8. **Q: How do you implement secure error handling in REST APIs?**
   **A:** Never expose sensitive information in error messages, use generic error responses for security-related failures, log detailed errors server-side, implement consistent error response formats, avoid stack traces in production, and provide helpful but safe error information to legitimate users.

9. **Q: What are the security implications of CORS and how do you configure it safely?**
   **A:** CORS relaxes same-origin policy, potentially allowing cross-site attacks. Configure specific allowed origins (avoid wildcards with credentials), limit allowed methods and headers, set appropriate preflight cache times, and never allow credentials with wildcard origins.

10. **Q: How do you implement comprehensive security monitoring for REST APIs?**
    **A:** Log all security events (authentication, authorization, input validation failures), monitor for anomalous patterns, implement real-time alerting, track API usage metrics, perform regular security audits, use SIEM systems for correlation, and maintain incident response procedures.
