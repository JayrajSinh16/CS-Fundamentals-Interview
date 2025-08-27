# HTTP/HTTPS

Brief: Hypertext Transfer Protocol (HTTP) is the foundation of web communication, enabling browsers and servers to exchange data. HTTPS adds security through TLS encryption. Understanding HTTP methods, status codes, headers, and security mechanisms is essential for web development and system design interviews.

## Detailed Theory

### HTTP (Hypertext Transfer Protocol)

#### Basic Concepts
- **Stateless Protocol**: Each request is independent, no connection state maintained
- **Request-Response Model**: Client sends request, server responds
- **ASCII-based**: Human-readable text protocol
- **Port 80**: Default port for HTTP
- **Application Layer**: Operates at Layer 7 of OSI model

#### HTTP Request Structure
```
Method URI HTTP-Version CRLF
Header-Name: Header-Value CRLF
Header-Name: Header-Value CRLF
CRLF
[Message Body]
```

Example:
```
GET /index.html HTTP/1.1
Host: www.example.com
User-Agent: Mozilla/5.0
Accept: text/html,application/xhtml+xml
Connection: keep-alive

```

#### HTTP Response Structure
```
HTTP-Version Status-Code Reason-Phrase CRLF
Header-Name: Header-Value CRLF
Header-Name: Header-Value CRLF
CRLF
[Message Body]
```

Example:
```
HTTP/1.1 200 OK
Content-Type: text/html
Content-Length: 1234
Server: Apache/2.4.41
Date: Mon, 27 Aug 2025 10:30:00 GMT

<html><body>Hello World</body></html>
```

### HTTP Methods

#### Safe and Idempotent Methods
- **Safe**: Don't modify server state (GET, HEAD, OPTIONS)
- **Idempotent**: Multiple identical requests have same effect (GET, PUT, DELETE, HEAD, OPTIONS)

#### Detailed Method Descriptions

**GET**
- Retrieve data from server
- Should not modify server state
- Can be cached
- Query parameters in URL
```
GET /api/users?page=1&limit=10 HTTP/1.1
Host: api.example.com
```

**POST**
- Submit data to server
- Can modify server state
- Data in request body
- Not idempotent
```
POST /api/users HTTP/1.1
Host: api.example.com
Content-Type: application/json

{"name": "John Doe", "email": "john@example.com"}
```

**PUT**
- Update or create resource
- Idempotent
- Complete replacement of resource
```
PUT /api/users/123 HTTP/1.1
Host: api.example.com
Content-Type: application/json

{"id": 123, "name": "John Smith", "email": "john.smith@example.com"}
```

**PATCH**
- Partial update of resource
- May or may not be idempotent
```
PATCH /api/users/123 HTTP/1.1
Host: api.example.com
Content-Type: application/json

{"email": "newemail@example.com"}
```

**DELETE**
- Remove resource
- Idempotent
```
DELETE /api/users/123 HTTP/1.1
Host: api.example.com
```

**HEAD**
- Same as GET but only returns headers
- Used for metadata checking
```
HEAD /large-file.zip HTTP/1.1
Host: downloads.example.com
```

**OPTIONS**
- Returns allowed methods for resource
- Used in CORS preflight requests
```
OPTIONS /api/users HTTP/1.1
Host: api.example.com
```

### HTTP Status Codes

#### 1xx Informational
- **100 Continue**: Server received request headers, client should send body
- **101 Switching Protocols**: Server switching to protocol specified in Upgrade header

#### 2xx Success
- **200 OK**: Request successful
- **201 Created**: Resource created successfully
- **202 Accepted**: Request accepted for processing
- **204 No Content**: Success but no content to return
- **206 Partial Content**: Partial resource delivered (range requests)

#### 3xx Redirection
- **301 Moved Permanently**: Resource permanently moved
- **302 Found**: Resource temporarily moved
- **304 Not Modified**: Resource not modified since last request
- **307 Temporary Redirect**: Temporary redirect, method must not change
- **308 Permanent Redirect**: Permanent redirect, method must not change

#### 4xx Client Error
- **400 Bad Request**: Malformed request
- **401 Unauthorized**: Authentication required
- **403 Forbidden**: Access denied
- **404 Not Found**: Resource not found
- **405 Method Not Allowed**: HTTP method not supported
- **409 Conflict**: Request conflicts with current resource state
- **422 Unprocessable Entity**: Valid request but semantic errors
- **429 Too Many Requests**: Rate limiting

#### 5xx Server Error
- **500 Internal Server Error**: Generic server error
- **501 Not Implemented**: Server doesn't support functionality
- **502 Bad Gateway**: Invalid response from upstream server
- **503 Service Unavailable**: Server temporarily unavailable
- **504 Gateway Timeout**: Timeout from upstream server

### Important HTTP Headers

#### Request Headers
```
Host: www.example.com                    # Required in HTTP/1.1
User-Agent: Mozilla/5.0                  # Client information
Accept: text/html,application/xhtml+xml  # Acceptable content types
Accept-Language: en-US,en                # Preferred languages
Accept-Encoding: gzip, deflate           # Supported compressions
Authorization: Bearer token123           # Authentication
Cookie: sessionid=abc123                 # Client state
Referer: https://google.com              # Previous page
Cache-Control: no-cache                  # Caching directives
```

#### Response Headers
```
Content-Type: text/html; charset=UTF-8   # Response content type
Content-Length: 1234                     # Response body size
Content-Encoding: gzip                   # Response compression
Set-Cookie: sessionid=xyz789             # Set client cookie
Location: https://example.com/new        # Redirect location
Cache-Control: max-age=3600              # Caching instructions
ETag: "123456789"                        # Resource version
Last-Modified: Wed, 21 Oct 2015 07:28:00 GMT
```

### HTTPS (HTTP Secure)

#### Overview
- HTTP over TLS/SSL
- Default port 443
- Provides confidentiality, integrity, and authentication
- Required for modern web security

#### TLS Handshake Process
1. **Client Hello**: Supported cipher suites, random number
2. **Server Hello**: Selected cipher suite, certificate, random number
3. **Certificate Verification**: Client validates server certificate
4. **Key Exchange**: Establish shared secret (ECDHE, RSA)
5. **Finished**: Both sides verify handshake integrity
6. **Application Data**: Encrypted HTTP communication

#### Security Benefits
- **Encryption**: Data protected from eavesdropping
- **Integrity**: Data cannot be tampered with
- **Authentication**: Server identity verified (client optional)
- **Forward Secrecy**: Past communications secure even if key compromised

### HTTP Versions

#### HTTP/1.0 (1996)
- Simple request-response
- New connection for each request
- No persistent connections

#### HTTP/1.1 (1997)
- Persistent connections (keep-alive)
- Pipelining (limited support)
- Chunked transfer encoding
- Host header required
- Cache improvements

#### HTTP/2 (2015)
- Binary protocol (not text-based)
- Multiplexing: Multiple requests over single connection
- Server push: Server can send resources proactively
- Header compression (HPACK)
- Stream prioritization
- Requires HTTPS in browsers

#### HTTP/3 (2022)
- Built on QUIC (UDP-based)
- Reduces connection setup time
- Better handling of packet loss
- Improved security and performance

### Practical Examples

#### RESTful API Design
```python
# Python Flask example
from flask import Flask, request, jsonify

app = Flask(__name__)

@app.route('/api/users', methods=['GET'])
def get_users():
    # GET /api/users - List users
    return jsonify({"users": []})

@app.route('/api/users', methods=['POST'])
def create_user():
    # POST /api/users - Create user
    data = request.json
    return jsonify({"id": 123, "name": data["name"]}), 201

@app.route('/api/users/<int:user_id>', methods=['GET'])
def get_user(user_id):
    # GET /api/users/123 - Get specific user
    return jsonify({"id": user_id, "name": "John Doe"})

@app.route('/api/users/<int:user_id>', methods=['PUT'])
def update_user(user_id):
    # PUT /api/users/123 - Update user
    data = request.json
    return jsonify({"id": user_id, "name": data["name"]})

@app.route('/api/users/<int:user_id>', methods=['DELETE'])
def delete_user(user_id):
    # DELETE /api/users/123 - Delete user
    return '', 204
```

#### CORS (Cross-Origin Resource Sharing)
```javascript
// Preflight request for complex requests
OPTIONS /api/users HTTP/1.1
Host: api.example.com
Origin: https://myapp.com
Access-Control-Request-Method: POST
Access-Control-Request-Headers: Content-Type

// Server response
HTTP/1.1 200 OK
Access-Control-Allow-Origin: https://myapp.com
Access-Control-Allow-Methods: GET, POST, PUT, DELETE
Access-Control-Allow-Headers: Content-Type
Access-Control-Max-Age: 86400
```

### Security Considerations

#### Common Vulnerabilities
- **Cross-Site Scripting (XSS)**: Malicious scripts in web pages
- **Cross-Site Request Forgery (CSRF)**: Unauthorized actions on behalf of user
- **SQL Injection**: Malicious SQL queries through input
- **Clickjacking**: Tricking users into clicking hidden elements

#### Security Headers
```
Strict-Transport-Security: max-age=31536000; includeSubDomains
Content-Security-Policy: default-src 'self'
X-Frame-Options: DENY
X-Content-Type-Options: nosniff
X-XSS-Protection: 1; mode=block
Referrer-Policy: strict-origin-when-cross-origin
```

## Interview Questions

1. **Q: Explain the difference between HTTP and HTTPS.**
   **A:** HTTP is plaintext and insecure; HTTPS adds TLS encryption for confidentiality, integrity, and authentication. HTTPS uses port 443 vs HTTP's port 80 and is required for modern web security.

2. **Q: What is the difference between GET and POST methods?**
   **A:** GET retrieves data, is idempotent and safe, can be cached, and puts parameters in URL. POST submits data, may modify server state, is not idempotent, and puts data in request body.

3. **Q: When would you use PUT vs PATCH for updates?**
   **A:** PUT replaces the entire resource and is idempotent. PATCH performs partial updates and may not be idempotent. Use PUT for complete replacement, PATCH for specific field updates.

4. **Q: Explain the most common HTTP status codes.**
   **A:** 200 (OK), 201 (Created), 400 (Bad Request), 401 (Unauthorized), 403 (Forbidden), 404 (Not Found), 500 (Internal Server Error), 502 (Bad Gateway), 503 (Service Unavailable).

5. **Q: What is HTTP caching and how does it work?**
   **A:** HTTP caching stores responses to reduce server load and improve performance. Uses headers like Cache-Control, ETag, Last-Modified, and Expires to control caching behavior.

6. **Q: How does HTTPS TLS handshake work?**
   **A:** Client Hello → Server Hello + Certificate → Key Exchange → Finished messages. Establishes encryption keys and verifies server identity before encrypted communication begins.

7. **Q: What is CORS and why is it needed?**
   **A:** Cross-Origin Resource Sharing allows web pages to access resources from different domains. Browsers enforce same-origin policy, CORS provides controlled relaxation with preflight requests for complex operations.

8. **Q: Explain the difference between cookies and sessions.**
   **A:** Cookies are client-side storage with size limits and security concerns. Sessions store data server-side with only session ID in cookie, providing better security and unlimited storage.

9. **Q: What are the main differences between HTTP/1.1 and HTTP/2?**
   **A:** HTTP/2 is binary (vs text), supports multiplexing (vs sequential), includes server push, header compression (HPACK), and stream prioritization. Requires HTTPS in browsers.

10. **Q: How do you secure HTTP APIs?**
    **A:** Use HTTPS, implement authentication (JWT, OAuth), validate input, use CORS properly, implement rate limiting, add security headers, and sanitize outputs to prevent XSS.

11. **Q: What is REST and what makes an API RESTful?**
    **A:** REST uses HTTP methods semantically, resources identified by URLs, stateless communication, and uniform interface. RESTful APIs follow these principles with proper HTTP status codes and HATEOAS.

12. **Q: How do you handle authentication in HTTP?**
    **A:** Basic Auth (base64 username:password), Bearer tokens (JWT), API keys, OAuth 2.0, or session-based authentication. Each has different security properties and use cases.
