# RESTful vs Non-RESTful Design

Designing RESTful APIs means embracing HTTP semantics, statelessness, and resource-oriented modeling. Non-RESTful designs often drift into RPC-like patterns or misuse HTTP.

## What Makes an API RESTful?

- Resource-oriented URLs (nouns) rather than actions (verbs)
- Proper use of HTTP methods (GET, POST, PUT, PATCH, DELETE)
- Stateless server interactions
- Standard HTTP status codes
- Cacheable responses when appropriate
- Content negotiation via Accept/Content-Type
- HATEOAS (optional but recommended for discoverability)

## RESTful vs Non-RESTful Examples

- URL Design
```http
# RESTful
GET /users/123
GET /users/123/posts
POST /users

# Non-RESTful (RPC-style)
GET /getUser?id=123
POST /createUser
POST /doActionOnUser
```

- Method Semantics
```http
# RESTful
PATCH /users/123  { "email": "new@ex.com" }
DELETE /users/123

# Non-RESTful
POST /users/updateEmail  { "id":123, "email": "new@ex.com" }
GET /deleteUser?id=123
```

- Status Codes
```http
# RESTful
POST /users  -> 201 Created (Location header)
GET /users/999 -> 404 Not Found

# Non-RESTful
POST /users -> 200 OK with custom code { "status": "created" }
GET /users/999 -> 200 OK with { "error": "not found" }
```

## Anti-Patterns

- Tunneling everything through POST
- Action verbs in paths: /approveOrder, /getUser
- Ignoring HTTP cache headers and ETags
- Custom status codes in body instead of HTTP codes
- Overloaded endpoints doing many different actions

## Pragmatic REST

- It's okay to bend rules when justified (e.g., complex workflows)
- Document deviations and rationale
- Keep consistency and predictability for clients

## Migration Tips

- Introduce resource-based endpoints alongside legacy RPC routes
- Map actions to sub-resources: /orders/{id}/approval
- Use 202 Accepted + job status for long-running operations
- Add proper status codes and headers incrementally

## Interview Questions

1. What makes an API “RESTful” in practice?
2. Show how you would refactor /createUser and /updateUser to REST.
3. When would you choose RPC over REST?
4. Why is statelessness important in REST?
5. How do you handle actions like “approve”, “cancel” with REST paths?
6. What are the downsides of action-based endpoints?
7. How do you evolve a non-RESTful API to REST without breaking clients?
8. Are HATEOAS and REST required together? Explain.
9. Give examples of cache-friendly REST patterns.
10. Compare PUT vs PATCH in a RESTful design.
