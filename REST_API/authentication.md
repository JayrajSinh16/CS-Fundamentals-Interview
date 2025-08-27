# Authentication Methods

APIs need to authenticate callers before granting access. Choose methods based on client type and risk.

## Common Methods

- API Keys: Simple, for server-to-server or public data with rate limiting
- Basic Auth: Legacy, only over HTTPS; avoid for modern APIs
- Bearer Tokens: JWT or opaque, sent in Authorization header
- OAuth 2.0: Delegated access with flows (Auth Code, Client Credentials)
- Mutual TLS: Strong client authentication via certificates

## Examples

```http
# API key (header preferred)
GET /api/data
X-API-Key: ak_live_123

# Basic Auth (base64 user:pass)
GET /api/me
Authorization: Basic am9objpzZWNyZXQ=

# Bearer token
GET /api/profile
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

# mTLS (TLS handshake validates client cert)
GET /api/secure
# Server requires client certificate trusted by CA
```

## Choosing

- Mobile/web apps: OAuth 2.0 + PKCE; short-lived access tokens, refresh tokens
- Service-to-service: OAuth 2.0 Client Credentials or mTLS
- Internal admin tools: SSO (OIDC) with RBAC

## Best Practices

- Always use HTTPS
- Short-lived tokens; rotate keys regularly
- Store secrets securely; never in client-side code
- Scope tokens/keys to least privilege
- Provide revocation and audit trails

## Interview Questions

1. API keys vs JWTs—when and why?
2. Which OAuth flows suit native apps vs SPAs?
3. How to rotate and revoke tokens safely?
4. How to secure tokens in browsers and mobile?
5. mTLS pros/cons vs OAuth?
