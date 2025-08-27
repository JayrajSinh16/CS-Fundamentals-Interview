# OAuth 2.0

OAuth 2.0 is an authorization framework enabling delegated access. Users grant an application limited access to resources without sharing credentials.

## Core Roles

- Resource Owner: End-user
- Client: Application requesting access
- Authorization Server: Issues tokens
- Resource Server: API accepting tokens

## Grants (Flows)

- Authorization Code + PKCE (public clients like SPAs/mobile)
- Client Credentials (machine-to-machine)
- Refresh Token (obtain new access tokens)
- Device Code (TVs/consoles)

## Authorization Code + PKCE

```http
# 1) Client initiates
GET https://auth.example.com/authorize?response_type=code&client_id=app&redirect_uri=https://app/cb&scope=read%20write&code_challenge=...&code_challenge_method=S256

# 2) User authenticates and consents
# 3) Redirect with code
GET https://app/cb?code=abc

# 4) Exchange code for tokens
POST https://auth.example.com/token
Content-Type: application/x-www-form-urlencoded

grant_type=authorization_code&code=abc&redirect_uri=https://app/cb&code_verifier=...

200 OK
{
  "access_token": "eyJ...",
  "expires_in": 3600,
  "refresh_token": "def...",
  "token_type": "Bearer",
  "scope": "read write"
}
```

## Scopes and Consent

- Design scopes per resource/action: read:users, write:posts
- Show clear consent screens; log consents

## Token Introspection and Revocation

- Introspection endpoint for opaque tokens
- Revocation endpoint for access/refresh tokens

## Security Tips

- Use PKCE for public clients, always HTTPS
- Rotate refresh tokens; detect reuse
- Limit scope and lifetime
- Use state and nonce to prevent CSRF/replay

## Interview Questions

1. Which OAuth flow for a mobile app and why?
2. How do PKCE and state improve security?
3. Token introspection vs JWT validation?
4. How do you design granular scopes?
5. How would you revoke tokens across microservices?
