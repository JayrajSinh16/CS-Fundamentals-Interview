# API vs Web Service

API and Web Service are related but distinct concepts. Understanding the difference helps in choosing architectures and communicating requirements precisely.

## Definitions

- API (Application Programming Interface)
  - A contract that allows software components to communicate
  - Language-agnostic, protocol-agnostic (can be local in-process, over HTTP, via RPC, etc.)
  - Examples: REST API over HTTP, OS APIs, library SDKs

- Web Service
  - A type of API accessible over a network using web protocols
  - Typically uses HTTP/HTTPS and standardized formats
  - Examples: RESTful web services, SOAP web services

## Key Differences

- Scope: All web services are APIs, but not all APIs are web services
- Transport: APIs can be in-process or over various protocols; web services are over the web (HTTP/etc.)
- Standards: SOAP web services have strict standards; REST web services embrace HTTP semantics
- Formats: APIs may use binary (gRPC/Protobuf), JSON, XML, etc.

## Examples

- API (non-web):
  - Java's Collections API (in-process method calls)
  - OS system call API
  - Database driver API (JDBC)

- Web Service (web API):
  - REST: GET /users/{id}
  - SOAP: POST /UserService with XML envelope
  - GraphQL: POST /graphql with query body

## Comparing Modern Options

- RESTful Web Services
  - Resource-oriented, HTTP methods
  - JSON payloads, status codes, caching

- SOAP Web Services
  - XML envelopes, WSDL contracts
  - WS-* standards, enterprise features

- gRPC (Web API with HTTP/2)
  - Binary Protobuf, strongly typed
  - Streaming, low latency, great for microservices

## Choosing the Right One

- Mobile/web clients: REST or GraphQL
- Inter-service communication: gRPC or REST
- Legacy/enterprise integrations: SOAP
- Browser caching/CDN needs: REST

## Interview Questions

1. Is every API a web service? Explain.
2. Compare REST, SOAP, and gRPC as web services.
3. When is a non-web API preferable?
4. How does network transport affect API design?
5. How do you secure APIs vs web services?
6. What trade-offs exist between gRPC and REST?
7. Give examples of APIs that are not web services.
8. How do you expose a non-web API as a web service?
9. What does protocol-agnostic API design mean?
10. How would you migrate SOAP services to REST?
