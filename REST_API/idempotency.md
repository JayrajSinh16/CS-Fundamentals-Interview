# Idempotency

Idempotency means multiple identical requests have the same effect as one request. It’s critical for reliability and retries.

## By HTTP Method

- GET, HEAD, OPTIONS: inherently idempotent
- PUT, DELETE: should be idempotent by design
- POST: not idempotent by default—use idempotency keys when needed

## Idempotency Keys

```http
POST /api/payments
Idempotency-Key: 2f0a6b58-0b97-4f9b-8b19-a1d7a7f8e6d4
Content-Type: application/json

{ "amount": 5000, "currency": "USD", "source": "tok_visa" }

201 Created
Idempotency-Key: 2f0a6b58-0b97-4f9b-8b19-a1d7a7f8e6d4
{
  "id": "pay_123",
  "status": "succeeded"
}

# Same request again
POST /api/payments
Idempotency-Key: 2f0a6b58-0b97-4f9b-8b19-a1d7a7f8e6d4

200 OK
{
  "id": "pay_123",
  "status": "succeeded",
  "idempotent": true
}
```

## Implementing

- Store request hash keyed by Idempotency-Key + endpoint + body
- Return same response for duplicates until expiry window
- Handle concurrency with locks

## When to Use

- Payments, orders, signup, email sending
- Any operation clients might retry

## Interview Questions

1. How to implement idempotency for POST?
2. Why is PUT idempotent but PATCH might not be?
3. How to handle idempotency storage and TTL?
4. What are race conditions to avoid?
5. How does idempotency interact with eventual consistency?
