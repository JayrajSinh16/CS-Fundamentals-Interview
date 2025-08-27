# Common cURL Commands

GET with query params and headers
```bash
curl -sS "https://api.example.com/orders?status=pending&page=2" \
  -H "Accept: application/json" \
  -H "Authorization: Bearer $TOKEN"
```

POST JSON
```bash
curl -sS -X POST "https://api.example.com/orders" \
  -H "Content-Type: application/json" \
  -d '{"items":[{"sku":"A1","qty":2}]}'
```

PUT with ETag (optimistic concurrency)
```bash
curl -sS -X PUT "https://api.example.com/orders/123" \
  -H "Content-Type: application/json" \
  -H "If-Match: \"v1-abc123\"" \
  -d '{"status":"confirmed"}'
```

PATCH
```bash
curl -sS -X PATCH "https://api.example.com/orders/123" \
  -H "Content-Type: application/merge-patch+json" \
  -d '{"status":"shipped"}'
```

DELETE
```bash
curl -sS -X DELETE "https://api.example.com/orders/123"
```

File upload (multipart)
```bash
curl -sS -X POST "https://api.example.com/files" \
  -F "file=@report.pdf" \
  -F "meta={\"type\":\"report\"};type=application/json"
```

Follow redirects and show headers
```bash
curl -sSIL https://api.example.com/health
```
