# Long Polling vs WebSockets vs Server-Sent Events (SSE)

Real-time communication patterns for delivering updates to clients.

## Long Polling

- Client requests and server holds connection until data or timeout
- Simple to implement, works everywhere
- Higher overhead; not true real-time under load

Example:
```http
GET /api/updates?cursor=abcd

200 OK
{
  "events": [ ... ],
  "next_cursor": "efgh"
}
```

## WebSockets

- Full-duplex, persistent connection over TCP (ws/wss)
- Best for bi-directional, low-latency messaging
- Requires stateful infrastructure / sticky sessions typically

Example (pseudo):
```text
ws = new WebSocket('wss://api.example.com/realtime');
ws.send(JSON.stringify({ subscribe: 'orders' }));
ws.onmessage = (msg) => { /* handle */ };
```

## Server-Sent Events (SSE)

- One-way server-to-client event stream over HTTP
- Simpler than WebSockets for push notifications
- Auto-reconnect, event IDs for resume

Example:
```http
GET /events
Accept: text/event-stream

200 OK
Content-Type: text/event-stream

event: order
id: 12345
data: {"status":"paid","order_id":"ord_1"}

retry: 5000
```

## Choosing

- Notifications/feeds: SSE
- Chat/gaming/collab: WebSockets
- Legacy/simple: long polling

## Interview Questions

1. When choose SSE over WebSockets?
2. How to scale WebSockets horizontally?
3. How to resume streams after disconnects?
4. How to secure and authenticate real-time connections?
5. Trade-offs of long polling under load?
