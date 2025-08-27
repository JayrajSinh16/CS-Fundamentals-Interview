# JWT & Token-Based Authentication

JSON Web Tokens (JWT) are a compact way to transmit claims between parties securely. Commonly used for stateless API authentication.

## Structure

- Header: alg, typ
- Payload: claims (sub, iss, exp, iat, scopes)
- Signature: HMAC/RSASSA over header.payload

Example JWT (header.payload.signature):
```json
{
  "alg": "RS256",
  "typ": "JWT"
}
.
{
  "iss": "https://auth.example.com",
  "sub": "user_123",
  "aud": "api://default",
  "exp": 1693140000,
  "scope": "read:users write:posts",
  "roles": ["user"]
}
```

## Using JWTs

```http
# Login to obtain tokens
POST /auth/login
{
  "username": "john@example.com",
  "password": "secret"
}

200 OK
{
  "access_token": "eyJ...",
  "token_type": "Bearer",
  "expires_in": 3600,
  "refresh_token": "def..."
}

# Call API with token
GET /api/profile
Authorization: Bearer eyJ...
```

## Validation Checklist

- Verify signature with correct algorithm
- Check exp, nbf, iat, iss, aud
- Enforce scopes/roles per endpoint
- Handle token revocation/blacklist if needed

## Refresh Tokens

- Keep long-lived refresh tokens secure (HttpOnly cookies)
- Rotate on each use; revoke on anomaly
- Scope refresh tokens narrowly

## Best Practices

- Prefer asymmetric signing (RS256) in distributed systems
- Short-lived access tokens (5–15 min)
- Store tokens securely (never in localStorage if possible)
- Implement token introspection for opaque tokens

## Interview Questions

1. How do you validate a JWT on the server?
2. Access vs refresh token lifetimes and storage?
3. How to implement logout with stateless JWTs?
4. What’s the risk of alg=none and how to prevent it?
5. When to choose opaque tokens over JWTs?
