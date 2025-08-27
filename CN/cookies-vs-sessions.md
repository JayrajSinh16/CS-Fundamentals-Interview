# Cookies vs Sessions

Brief: Maintain state over stateless HTTP.

## Key concepts
- Cookies: client-stored key/value; attributes (Secure, HttpOnly, SameSite)
- Sessions: server-side state keyed by cookie; or stateless JWTs
- CSRF protection with SameSite or tokens

## Practical example
- Use HttpOnly+Secure cookies for session IDs; rotate on login.

## Common interview Q&A
- JWT pros/cons? How to prevent session fixation?
