# REST in Microservices

REST is commonly used for communication between microservices and between clients and the system.

## Internal vs External APIs

- External (north-south): stable, versioned, consumer-friendly
- Internal (east-west): optimized, smaller contracts, may use gRPC

## Patterns

- API Gateway at the edge
- Service decomposition by bounded contexts
- Synchronous REST with retries + circuit breaker
- Async messaging for events (outbox, saga)

## Resilience

- Timeouts, retries with backoff, jitter
- Circuit breaker and bulkheads
- Idempotency and deduplication

## Data Ownership

- Each service owns its data
- Avoid distributed transactions; use eventual consistency
- Patterns: Saga, CQRS, event sourcing

## Observability

- Correlation IDs across services
- Distributed tracing (W3C Trace Context)
- Centralized logging + metrics

## Interview Questions

1. Sync vs async between microservices—when and why?
2. How to avoid tight coupling between services?
3. Saga pattern—how to implement with REST endpoints?
4. How do you handle partial failures across services?
5. Versioning and compatibility strategies internally vs externally?
