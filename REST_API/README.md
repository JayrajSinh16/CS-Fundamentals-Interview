# REST API Interview Preparation Guide

Comprehensive, interview-focused guide covering REST API concepts, design patterns, and practical implementation. Each topic includes examples, best practices, and interview questions commonly asked at tech companies.

## REST API Fundamentals
- [REST Principles & Constraints](./rest-principles.md)
- [REST vs SOAP vs GraphQL](./rest-vs-soap.md)
- [API vs Web Service](./api-vs-webservice.md)
- [RESTful vs Non-RESTful Design](./restful-design.md)

## HTTP Fundamentals
- [HTTP Methods & CRUD Mapping](./http-methods.md)
- [HTTP Status Codes](./http-status-codes.md)
- [HTTP Headers](./http-headers.md)
- [Request/Response Structure](./request-response.md)

## REST Design Principles
- [Resource-Based URL Design](./url-design.md)
- [API Versioning Strategies](./api-versioning.md)
- [HATEOAS (Hypermedia)](./hateoas.md)
- [Path vs Query Parameters](./parameters.md)

## Data Formats & Content Negotiation
- [JSON vs XML](./json-vs-xml.md)
- [Content-Type & Accept Headers](./content-negotiation.md)
- [Request/Response Body Structure](./request-response-body.md)
- [Data Validation](./data-validation.md)

## Authentication & Security
- [Authentication Methods](./authentication.md)
- [JWT & Token-Based Auth](./jwt-tokens.md)
- [OAuth 2.0](./oauth2.md)
- [API Security Best Practices](./security-best-practices.md)
- [CORS Handling](./cors.md)

## Best Practices & Design Patterns
- [Idempotency](./idempotency.md)
- [Pagination Techniques](./pagination-strategies.md)
- [Filtering & Sorting](./filtering-sorting.md)
- [Error Handling Patterns](./error-handling.md)
- [Rate Limiting](./rate-limiting.md)

## Advanced Topics
- [Caching Strategies](./caching-performance.md)
- [Webhooks](./webhooks.md)
- [Long Polling vs WebSockets vs SSE](./real-time-communication.md)
- [API Gateway](./api-gateway.md)
- [REST in Microservices](./microservices.md)

## Implementation & Testing
- [API Documentation (OpenAPI/Swagger)](./api-documentation.md)
- [Testing with Postman/cURL](./api-testing.md)
- [Mock APIs](./mock-apis.md)
- [API Monitoring](./api-monitoring.md)

## Common Patterns & Examples
- [Complete CRUD Operations](./crud-examples.md)
- [Bulk Operations](./bulk-operations.md)
- [Async Operations (202 Pattern)](./async-operations.md)
- [File Upload/Download](./file-operations.md)
- [Search & Complex Queries](./search-queries.md)

## Quick Reference
- [HTTP Methods Cheat Sheet](./http-methods-cheat.md)
- [Status Codes Quick Reference](./status-codes-quick.md)
- [Common cURL Commands](./curl-examples.md)
- [Interview Questions Bank](./interview-questions.md)

---

## How to Use This Guide

1. **Start with Fundamentals** - Understand REST principles and HTTP basics
2. **Practice Design** - Work through URL design and API patterns
3. **Implement Security** - Learn authentication and security best practices
4. **Advanced Concepts** - Explore caching, real-time communication, and microservices
5. **Test Knowledge** - Use interview questions to validate understanding

## Interview Tips

- **Think Resource-First**: Always design APIs around resources, not actions
- **Use HTTP Semantics**: Map CRUD to appropriate HTTP methods
- **Handle Errors Gracefully**: Provide meaningful error messages and status codes
- **Security by Design**: Always consider authentication, authorization, and data protection
- **Design for Scale**: Consider pagination, caching, and rate limiting from the start
