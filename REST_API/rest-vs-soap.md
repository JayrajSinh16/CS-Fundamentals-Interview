# REST vs SOAP vs GraphQL

Understanding the differences between REST, SOAP, and GraphQL helps you choose the right API style for your use case. Each has strengths, trade-offs, and ideal scenarios.

## At a Glance

- REST: Resource-oriented, HTTP-based, flexible, widely adopted
- SOAP: Protocol with strict standards, WS-* features, enterprise integration
- GraphQL: Query language with single endpoint, client-driven data fetching

## REST

- Architecture style using HTTP verbs, URIs, status codes
- Resource-based modeling (nouns) and representations (JSON/HTML/XML)
- Stateless, cache-friendly, simple and scalable

Example:
```http
GET /api/v1/users/123
Accept: application/json

200 OK
{
  "id": 123,
  "name": "John",
  "email": "john@example.com"
}
```

Pros:
- Simplicity, HTTP native concepts
- Caching and CDN friendly
- Loose coupling, microservices friendly
- Great tooling and ecosystem

Cons:
- Over/under-fetching possible
- Multiple round-trips for composite views
- Versioning management needed

## SOAP

- Protocol using XML envelopes and strict contracts (WSDL)
- Built-in standards: WS-Security, WS-AtomicTransaction, WS-ReliableMessaging
- Often over HTTP but not limited to it

Example SOAP Request:
```xml
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:usr="http://example.com/users">
  <soapenv:Header/>
  <soapenv:Body>
    <usr:GetUserRequest>
      <usr:Id>123</usr:Id>
    </usr:GetUserRequest>
  </soapenv:Body>
</soapenv:Envelope>
```

Pros:
- Strong contracts (WSDL), strict typing
- Advanced enterprise features (security, reliability, transactions)
- Good for legacy/enterprise integrations

Cons:
- Verbose XML, steeper learning curve
- Less suited for browser/mobile clients
- Harder caching, heavier payloads

## GraphQL

- Query language with schema and single endpoint (POST /graphql)
- Clients specify exactly the fields they need
- Strongly typed schema, introspection, powerful tooling

Example:
```http
POST /graphql
Content-Type: application/json

{
  "query": "query($id: ID!) { user(id: $id) { id name posts(limit: 2) { id title } } }",
  "variables": { "id": 123 }
}

200 OK
{
  "data": {
    "user": {
      "id": 123,
      "name": "John",
      "posts": [
        { "id": 1, "title": "Hello" },
        { "id": 2, "title": "World" }
      ]
    }
  }
}
```

Pros:
- Prevents over/under-fetching
- Single endpoint, fewer round-trips
- Strong typing and tooling (GraphiQL, codegen)

Cons:
- Caching more complex (though persisted queries + CDN help)
- N+1 query pitfalls without dataloaders
- Learning curve and server complexity

## When to Choose What

- Choose REST for public APIs, CRUD resources, cache-friendly endpoints
- Choose SOAP for enterprise systems needing WS-* features and strict contracts
- Choose GraphQL for mobile/web clients needing optimized data fetching and complex views

## Interoperability Tips

- REST + GraphQL hybrid: expose GraphQL internally, REST externally
- REST gateway in front of SOAP services for modern clients
- Use BFF (Backend For Frontend) patterns to tailor APIs per client

## Interview Questions

1. When would you choose GraphQL over REST?
2. How does caching differ between REST and GraphQL?
3. What advantages does SOAP offer in enterprise scenarios?
4. How do you prevent N+1 issues in GraphQL?
5. Compare versioning strategies in REST vs GraphQL.
6. Can REST be strongly typed like GraphQL? How?
7. How do you secure SOAP vs REST vs GraphQL endpoints?
8. How would you wrap a SOAP service to present a REST API?
9. What are typical performance pitfalls in each style?
10. How do you evolve schemas/contracts over time?
