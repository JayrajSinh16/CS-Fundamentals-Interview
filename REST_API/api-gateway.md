# API Gateway

An API Gateway is a reverse proxy that routes, secures, and manages API traffic for microservices.

## Responsibilities

- Routing and load balancing
- Authentication and authorization
- Rate limiting and quotas
- Caching and compression
- Request/response transformation
- Observability: logging, tracing, metrics

## Patterns

- Edge gateway for external clients
- Internal gateway for service-to-service
- BFF (Backend for Frontend) per client type

## Example Policy (pseudo-YAML)

```yaml
routes:
  - path: /api/users/**
    upstream: http://users-svc:8080
    auth: oauth2
    rate_limit: 1000/hour
    cache:
      enabled: true
      ttl: 60s
    transform:
      request:
        headers:
          add:
            X-Request-Id: ${uuid}
      response:
        remove_headers: ["Server", "X-Powered-By"]
```

## Gateway vs Service Mesh

- Gateway: north-south traffic
- Mesh: east-west traffic between services (sidecars)
- Often used together

## Interview Questions

1. Why use an API Gateway in microservices?
2. How do you implement auth at the gateway vs services?
3. Where to place rate limiting—gateway or service?
4. How to handle canary releases via gateway?
5. Compare gateway to service mesh features.
