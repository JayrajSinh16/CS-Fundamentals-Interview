# Webhooks

Webhooks let your API notify clients of events via HTTP callbacks.

## Design

- Event types: user.created, order.paid, invoice.failed
- Subscriptions: register callback URL and events
- Retries with exponential backoff
- Signature verification (HMAC)

## Delivery Example

```http
POST https://client.example.com/webhooks
Content-Type: application/json
X-Signature: t=1693126800,v1=hex(hmac_sha256(secret, payload))
X-Event-Type: order.paid

{
  "id": "evt_123",
  "type": "order.paid",
  "data": { "order_id": "ord_999", "amount": 5000 }
}

# Client responds
200 OK
```

## Retry

- 2xx = success
- Non-2xx: retry with backoff; stop after N attempts
- Idempotency key for webhook deliveries

## Security

- Verify signatures and timestamps
- Rotate secrets
- Use allow-listed IPs or mTLS for high-security

## Management

- Dashboard to view deliveries, replays
- Dead letter queue for repeated failures

## Interview Questions

1. How to ensure webhook reliability and idempotency?
2. How do you secure webhooks?
3. How to handle client downtime gracefully?
4. What admin tools help troubleshoot webhooks?
5. Compare polling vs webhooks vs SSE.
