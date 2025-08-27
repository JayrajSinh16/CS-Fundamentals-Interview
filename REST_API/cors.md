# CORS Handling

Cross-Origin Resource Sharing (CORS) controls which origins can call your API from browsers.

## Basics

- Preflight: Browser sends OPTIONS with Access-Control-Request-Method/Headers
- Server responds with Access-Control-Allow-* headers
- Simple requests (GET/POST with simple headers) may skip preflight

## Example

```http
# Preflight
OPTIONS /api/users
Origin: https://app.example.com
Access-Control-Request-Method: POST
Access-Control-Request-Headers: Authorization, Content-Type

204 No Content
Access-Control-Allow-Origin: https://app.example.com
Access-Control-Allow-Methods: GET, POST, PUT, PATCH, DELETE
Access-Control-Allow-Headers: Authorization, Content-Type
Access-Control-Allow-Credentials: true
Access-Control-Max-Age: 86400

# Actual request
POST /api/users
Origin: https://app.example.com
Authorization: Bearer eyJ...
Content-Type: application/json

201 Created
Access-Control-Allow-Origin: https://app.example.com
Access-Control-Allow-Credentials: true
```

## Best Practices

- Don’t use wildcard with credentials; list allowed origins
- Cache preflight with Max-Age
- Validate allowed methods and headers strictly
- Return Vary: Origin for proxies/CDNs

## Common Pitfalls

- Missing OPTIONS route handling
- Mismatch in allowed headers vs actual headers
- Allowing all origins blindly

## Interview Questions

1. How does the CORS preflight work?
2. Why avoid Access-Control-Allow-Origin: * with cookies?
3. How do you configure CORS dynamically per tenant?
4. What headers must be echoed back?
5. How to debug CORS issues in the browser?
