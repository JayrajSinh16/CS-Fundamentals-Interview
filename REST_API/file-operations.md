# File Upload/Download

Handle files efficiently and securely in REST APIs.

## Upload

- Use multipart/form-data for browser uploads
- For large files, use pre-signed URLs (S3/GCS) and direct-to-storage

```http
POST /api/files
Content-Type: multipart/form-data

--boundary
Content-Disposition: form-data; name="file"; filename="report.pdf"
Content-Type: application/pdf

<binary data>
--boundary--

201 Created
Location: /api/files/file_123
{
  "id": "file_123",
  "name": "report.pdf",
  "size": 234567,
  "content_type": "application/pdf"
}
```

## Download

- Use Content-Disposition for filename
- Support range requests (206 Partial Content) for large files
- Set Cache-Control and integrity headers

```http
GET /api/files/file_123/download

200 OK
Content-Type: application/pdf
Content-Disposition: attachment; filename="report.pdf"
Content-Length: 234567
```

## Integrity and Security

- Virus scan, content-type verification
- Size limits and quotas
- Signed URLs with expiry

## Interview Questions

1. Multipart vs direct-to-storage trade-offs?
2. How to resume interrupted downloads/uploads?
3. How to secure file access per user/role?
4. How to serve large files efficiently (CDN, ranges)?
5. Metadata and lifecycle management best practices?
