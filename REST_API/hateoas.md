# HATEOAS (Hypermedia as the Engine of Application State)

HATEOAS enriches responses with links that tell clients what they can do next, reducing hardcoded client logic and improving discoverability.

## Why it matters
- Decouples clients from URL structures
- Encourages workflow guidance via link relations (rel)
- Enables evolvable APIs without breaking clients

## Minimal example
```json
{
  "id": 123,
  "status": "pending",
  "_links": {
    "self": { "href": "/orders/123" },
    "customer": { "href": "/customers/42" },
    "cancel": { "href": "/orders/123/cancel", "method": "POST" },
    "pay": { "href": "/orders/123/payment", "method": "POST" }
  }
}
```

## Design tips
- Include stable rel names (self, next, prev, cancel, pay) and document them
- Provide method hints and, optionally, content types
- Keep links minimal; avoid spamming every related resource

## Trade-offs
- Extra payload size and implementation effort
- Many public APIs rely on docs over hypermedia; HATEOAS shines for complex workflows and internal clients

## Interview questions
1. What problems does HATEOAS solve, and when is it overkill?
2. How do you version and document link relations?
3. Compare HATEOAS with out-of-band API documentation.
4. How would you test presence and correctness of hypermedia links?
