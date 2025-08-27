# Content Negotiation

Content negotiation is a mechanism that allows clients and servers to agree on the most suitable representation of a resource. It enables APIs to serve the same resource in different formats, languages, encodings, or character sets based on client preferences. This flexibility is crucial for building APIs that can serve diverse clients with varying capabilities and requirements.

## What is Content Negotiation?

Content negotiation allows a single URL to serve multiple representations of the same resource. The client specifies its preferences through HTTP headers, and the server responds with the most appropriate representation or returns a 406 Not Acceptable status if it cannot satisfy the request.

### Basic Example
```http
# Client requests JSON
GET /api/users/123
Accept: application/json

Response: 200 OK
Content-Type: application/json
{
  "id": 123,
  "name": "John Doe",
  "email": "john@example.com"
}

# Client requests XML
GET /api/users/123
Accept: application/xml

Response: 200 OK
Content-Type: application/xml
<?xml version="1.0"?>
<user>
  <id>123</id>
  <name>John Doe</name>
  <email>john@example.com</email>
</user>

# Client requests unsupported format
GET /api/users/123
Accept: application/yaml

Response: 406 Not Acceptable
Content-Type: application/json
{
  "error": "not_acceptable",
  "message": "Requested format not available",
  "available_formats": ["application/json", "application/xml", "text/csv"]
}
```

## Types of Content Negotiation

### 1. Media Type Negotiation
Negotiating the format of the response content.

```http
# Multiple acceptable formats with quality values
GET /api/products
Accept: application/json,application/xml;q=0.8,text/csv;q=0.6,*/*;q=0.1

# Server chooses best available format
Response: 200 OK
Content-Type: application/json; charset=utf-8
{
  "products": [
    {"id": 1, "name": "Laptop", "price": 999.99},
    {"id": 2, "name": "Phone", "price": 599.99}
  ]
}

# CSV format request
GET /api/products
Accept: text/csv

Response: 200 OK
Content-Type: text/csv; charset=utf-8
Content-Disposition: attachment; filename="products.csv"

id,name,price
1,Laptop,999.99
2,Phone,599.99
```

### 2. Language Negotiation
Serving content in different languages.

```http
# Request English content
GET /api/articles/123
Accept-Language: en-US,en;q=0.9

Response: 200 OK
Content-Language: en-US
Content-Type: application/json

{
  "id": 123,
  "title": "Getting Started with REST APIs",
  "content": "REST APIs are a fundamental part of modern web development...",
  "author": "John Doe",
  "published_date": "2025-08-27"
}

# Request Spanish content
GET /api/articles/123
Accept-Language: es-ES,es;q=0.8,en;q=0.6

Response: 200 OK
Content-Language: es-ES
Content-Type: application/json

{
  "id": 123,
  "title": "Comenzando con APIs REST",
  "content": "Las APIs REST son una parte fundamental del desarrollo web moderno...",
  "author": "John Doe",
  "published_date": "2025-08-27"
}

# Language not available, fallback to default
GET /api/articles/123
Accept-Language: zh-CN

Response: 200 OK
Content-Language: en-US
Vary: Accept-Language
Content-Type: application/json

{
  "id": 123,
  "title": "Getting Started with REST APIs",
  "content": "REST APIs are a fundamental part of modern web development...",
  "warning": "Requested language not available, serving default English content"
}
```

### 3. Encoding Negotiation
Negotiating compression and encoding formats.

```http
# Request compressed response
GET /api/large-dataset
Accept-Encoding: gzip, deflate, br

Response: 200 OK
Content-Encoding: gzip
Content-Type: application/json
Content-Length: 1234
Vary: Accept-Encoding

# Compressed JSON response body

# Request without compression support
GET /api/large-dataset
Accept-Encoding: identity

Response: 200 OK
Content-Type: application/json
Content-Length: 5678

# Uncompressed JSON response body
```

### 4. Character Set Negotiation
Specifying preferred character encodings.

```http
# Request UTF-8 encoding
GET /api/international-data
Accept-Charset: utf-8,iso-8859-1;q=0.5

Response: 200 OK
Content-Type: application/json; charset=utf-8

{
  "message": "Hello 世界 🌍",
  "names": ["José", "François", "Müller", "王小明"]
}

# Request specific character set
GET /api/data
Accept-Charset: iso-8859-1

Response: 200 OK
Content-Type: application/json; charset=iso-8859-1

{
  "message": "Hello World",
  "note": "Special characters converted to compatible format"
}
```

## Quality Values (q-values)

Quality values indicate the relative preference for different options, ranging from 0.0 to 1.0.

```http
# Preference order: JSON (highest), XML (medium), CSV (low), anything else (lowest)
GET /api/data
Accept: application/json;q=1.0,application/xml;q=0.8,text/csv;q=0.6,*/*;q=0.1

# Language preferences: US English (highest), general English (medium), Spanish (low)
GET /api/content
Accept-Language: en-US;q=1.0,en;q=0.8,es;q=0.5

# Encoding preferences: brotli (highest), gzip (medium), deflate (low)
GET /api/files
Accept-Encoding: br;q=1.0,gzip;q=0.8,deflate;q=0.6

# Complex negotiation example
GET /api/products
Accept: application/json;q=0.9,application/xml;q=0.8,text/html;q=0.7
Accept-Language: en-US;q=1.0,en;q=0.8,fr;q=0.6
Accept-Encoding: gzip;q=1.0,deflate;q=0.8
Accept-Charset: utf-8;q=1.0,iso-8859-1;q=0.8

Response: 200 OK
Content-Type: application/json; charset=utf-8
Content-Language: en-US
Content-Encoding: gzip
Vary: Accept, Accept-Language, Accept-Encoding, Accept-Charset
```

## Server-Driven vs Agent-Driven Negotiation

### Server-Driven Negotiation
Server automatically selects the best representation based on request headers.

```http
# Client sends preferences
GET /api/users/123
Accept: application/json,application/xml;q=0.8
Accept-Language: en-US,es;q=0.6

# Server chooses and responds
Response: 200 OK
Content-Type: application/json; charset=utf-8
Content-Language: en-US
Vary: Accept, Accept-Language

{
  "id": 123,
  "name": "John Doe",
  "email": "john@example.com"
}
```

### Agent-Driven Negotiation
Server provides a list of available representations, client chooses.

```http
# Initial request
GET /api/users/123
Accept: */*

# Server responds with options
Response: 300 Multiple Choices
Content-Type: application/json

{
  "message": "Multiple representations available",
  "alternatives": [
    {
      "href": "/api/users/123",
      "type": "application/json",
      "language": "en-US",
      "title": "User data in JSON format"
    },
    {
      "href": "/api/users/123",
      "type": "application/xml", 
      "language": "en-US",
      "title": "User data in XML format"
    },
    {
      "href": "/api/users/123",
      "type": "application/json",
      "language": "es-ES",
      "title": "Datos del usuario en formato JSON"
    }
  ]
}

# Client makes specific choice
GET /api/users/123
Accept: application/xml
Accept-Language: en-US

Response: 200 OK
Content-Type: application/xml; charset=utf-8
Content-Language: en-US
```

## API Versioning Through Content Negotiation

### Version in Accept Header
```http
# Request specific API version
GET /api/users/123
Accept: application/vnd.myapi.v1+json

Response: 200 OK
Content-Type: application/vnd.myapi.v1+json

{
  "id": 123,
  "name": "John Doe",
  "email": "john@example.com"
}

# Request newer API version
GET /api/users/123
Accept: application/vnd.myapi.v2+json

Response: 200 OK
Content-Type: application/vnd.myapi.v2+json

{
  "id": 123,
  "full_name": "John Doe",
  "email_address": "john@example.com",
  "profile": {
    "created_at": "2025-01-15T10:30:00Z",
    "last_login": "2025-08-27T09:15:00Z"
  }
}

# Version not supported
GET /api/users/123
Accept: application/vnd.myapi.v5+json

Response: 406 Not Acceptable
{
  "error": "version_not_supported",
  "message": "API version 5 is not supported",
  "supported_versions": ["v1", "v2", "v3"],
  "latest_version": "v3"
}
```

### Vendor-Specific Media Types
```http
# GitHub API style versioning
GET /api/repositories/123
Accept: application/vnd.github.v3+json

Response: 200 OK
Content-Type: application/vnd.github.v3+json

# Stripe API style
GET /api/charges/123
Accept: application/vnd.stripe.v2020-08-27+json

Response: 200 OK
Content-Type: application/vnd.stripe.v2020-08-27+json
```

## Implementing Content Negotiation

### Server Implementation Example
```python
# Python Flask example
from flask import Flask, request, jsonify, make_response
import json
import xml.etree.ElementTree as ET

app = Flask(__name__)

@app.route('/api/users/<int:user_id>')
def get_user(user_id):
    user_data = {
        "id": user_id,
        "name": "John Doe",
        "email": "john@example.com"
    }
    
    # Parse Accept header
    accept_header = request.headers.get('Accept', 'application/json')
    
    if 'application/json' in accept_header:
        response = make_response(jsonify(user_data))
        response.headers['Content-Type'] = 'application/json'
        return response
    
    elif 'application/xml' in accept_header:
        # Convert to XML
        root = ET.Element('user')
        for key, value in user_data.items():
            elem = ET.SubElement(root, key)
            elem.text = str(value)
        
        xml_string = ET.tostring(root, encoding='unicode')
        response = make_response(xml_string)
        response.headers['Content-Type'] = 'application/xml'
        return response
    
    elif 'text/csv' in accept_header:
        # Convert to CSV
        csv_data = "id,name,email\n"
        csv_data += f"{user_data['id']},{user_data['name']},{user_data['email']}"
        
        response = make_response(csv_data)
        response.headers['Content-Type'] = 'text/csv'
        response.headers['Content-Disposition'] = 'attachment; filename="user.csv"'
        return response
    
    else:
        # Not acceptable
        error_response = {
            "error": "not_acceptable",
            "message": "Requested format not available",
            "available_formats": ["application/json", "application/xml", "text/csv"]
        }
        response = make_response(jsonify(error_response), 406)
        return response

# Add Vary header for caching
@app.after_request
def after_request(response):
    response.headers['Vary'] = 'Accept, Accept-Language, Accept-Encoding'
    return response
```

### Client Implementation Example
```javascript
// JavaScript client with content negotiation
class ApiClient {
  constructor(baseUrl) {
    this.baseUrl = baseUrl;
    this.defaultHeaders = {
      'Accept': 'application/json',
      'Accept-Language': 'en-US,en;q=0.9',
      'Accept-Encoding': 'gzip, deflate, br'
    };
  }

  async request(url, options = {}) {
    const headers = {
      ...this.defaultHeaders,
      ...options.headers
    };

    const response = await fetch(`${this.baseUrl}${url}`, {
      ...options,
      headers
    });

    if (response.status === 406) {
      const errorData = await response.json();
      throw new Error(`Content negotiation failed: ${errorData.message}`);
    }

    return this.parseResponse(response);
  }

  async parseResponse(response) {
    const contentType = response.headers.get('Content-Type');
    
    if (contentType.includes('application/json')) {
      return await response.json();
    } else if (contentType.includes('application/xml')) {
      const text = await response.text();
      // Parse XML (using DOMParser or xml2js library)
      const parser = new DOMParser();
      return parser.parseFromString(text, 'application/xml');
    } else if (contentType.includes('text/csv')) {
      return await response.text();
    } else {
      return await response.text();
    }
  }

  // Get user data in different formats
  async getUserAsJson(userId) {
    return this.request(`/api/users/${userId}`, {
      headers: { 'Accept': 'application/json' }
    });
  }

  async getUserAsXml(userId) {
    return this.request(`/api/users/${userId}`, {
      headers: { 'Accept': 'application/xml' }
    });
  }

  async getUserAsCsv(userId) {
    return this.request(`/api/users/${userId}`, {
      headers: { 'Accept': 'text/csv' }
    });
  }

  // API versioning through Accept header
  async getUserV2(userId) {
    return this.request(`/api/users/${userId}`, {
      headers: { 'Accept': 'application/vnd.myapi.v2+json' }
    });
  }
}
```

## Caching and Content Negotiation

### Vary Header
The Vary header tells caches which request headers affect the response content.

```http
# Request with specific preferences
GET /api/users/123
Accept: application/json
Accept-Language: en-US
Accept-Encoding: gzip

Response: 200 OK
Content-Type: application/json; charset=utf-8
Content-Language: en-US
Content-Encoding: gzip
Vary: Accept, Accept-Language, Accept-Encoding
Cache-Control: public, max-age=3600

# Different request with different preferences
GET /api/users/123
Accept: application/xml
Accept-Language: es-ES
Accept-Encoding: deflate

Response: 200 OK
Content-Type: application/xml; charset=utf-8
Content-Language: es-ES
Content-Encoding: deflate
Vary: Accept, Accept-Language, Accept-Encoding
Cache-Control: public, max-age=3600

# Cache will store separate copies for different Accept combinations
```

### ETags with Content Negotiation
```http
# First request
GET /api/users/123
Accept: application/json
Accept-Language: en-US

Response: 200 OK
Content-Type: application/json; charset=utf-8
Content-Language: en-US
ETag: "user-123-json-en-v1"
Vary: Accept, Accept-Language

# Conditional request
GET /api/users/123
Accept: application/json
Accept-Language: en-US
If-None-Match: "user-123-json-en-v1"

Response: 304 Not Modified
ETag: "user-123-json-en-v1"
Vary: Accept, Accept-Language

# Different format request
GET /api/users/123
Accept: application/xml
Accept-Language: en-US
If-None-Match: "user-123-json-en-v1"

Response: 200 OK
Content-Type: application/xml; charset=utf-8
Content-Language: en-US
ETag: "user-123-xml-en-v1"
Vary: Accept, Accept-Language
```

## Content Negotiation Best Practices

### 1. Provide Sensible Defaults
```http
# Client doesn't specify preferences
GET /api/users/123

# Server uses sensible defaults
Response: 200 OK
Content-Type: application/json; charset=utf-8
Content-Language: en-US
Vary: Accept, Accept-Language

{
  "id": 123,
  "name": "John Doe",
  "email": "john@example.com"
}
```

### 2. Support Standard Media Types
```http
# Support widely-used formats
GET /api/data
Accept: application/json     # Primary format
Accept: application/xml      # Alternative format
Accept: text/csv            # Export format
Accept: application/pdf     # Report format
Accept: text/html           # Browser-friendly format
```

### 3. Clear Error Messages
```http
# Helpful error response
GET /api/users/123
Accept: application/yaml

Response: 406 Not Acceptable
Content-Type: application/json

{
  "error": "not_acceptable",
  "message": "The requested format 'application/yaml' is not supported for this resource",
  "requested_format": "application/yaml",
  "available_formats": [
    {
      "format": "application/json",
      "description": "JSON representation"
    },
    {
      "format": "application/xml", 
      "description": "XML representation"
    },
    {
      "format": "text/csv",
      "description": "CSV export format"
    }
  ],
  "documentation": "https://api.example.com/docs/content-negotiation"
}
```

### 4. Performance Considerations
```http
# Optimize for common cases
GET /api/users
Accept: application/json    # Fast path - pre-serialized
Accept: application/xml     # Transform from JSON
Accept: text/csv           # Generate on-demand for exports

# Use caching effectively
Response: 200 OK
Content-Type: application/json
Cache-Control: public, max-age=3600, stale-while-revalidate=86400
Vary: Accept, Accept-Language
```

## Common Pitfalls

### 1. Ignoring Accept Header
```http
# Bad - Always return JSON regardless of Accept header
GET /api/users/123
Accept: application/xml

Response: 200 OK
Content-Type: application/json
# Returns JSON even though XML was requested

# Good - Honor Accept header or return 406
Response: 406 Not Acceptable
# Or transform to XML as requested
```

### 2. Incomplete Vary Header
```http
# Bad - Missing Vary header
GET /api/users/123
Accept: application/json
Accept-Language: en-US

Response: 200 OK
Content-Type: application/json; charset=utf-8
Content-Language: en-US
# Missing: Vary: Accept, Accept-Language

# Good - Include Vary header for caching
Response: 200 OK
Content-Type: application/json; charset=utf-8
Content-Language: en-US
Vary: Accept, Accept-Language
```

### 3. Poor Quality Value Handling
```http
# Bad - Not respecting q-values
GET /api/data
Accept: application/xml;q=0.1,application/json;q=0.9

Response: 200 OK
Content-Type: application/xml
# Should prefer JSON (higher q-value)

# Good - Respect quality preferences
Response: 200 OK
Content-Type: application/json
```

## Interview Questions

1. **Q: What is content negotiation and why is it important?**
   **A:** Content negotiation allows a single URL to serve multiple representations (formats, languages, encodings) of the same resource. It's important for API flexibility, supporting diverse clients, internationalization, and maintaining clean URL structures while serving different content types.

2. **Q: How do quality values (q-values) work in content negotiation?**
   **A:** Q-values (0.0-1.0) indicate preference order in Accept headers. Higher values mean higher preference. Example: `Accept: application/json;q=0.9,application/xml;q=0.6` prefers JSON over XML. Default q-value is 1.0.

3. **Q: What's the purpose of the Vary header in content negotiation?**
   **A:** Vary tells caches which request headers affect the response content. Example: `Vary: Accept, Accept-Language` means caches must consider both headers when storing/serving cached responses. Without it, wrong cached responses might be served.

4. **Q: How can content negotiation be used for API versioning?**
   **A:** Use vendor-specific media types in Accept header: `Accept: application/vnd.myapi.v2+json`. This keeps URLs clean while allowing version-specific representations. Better than URL-based versioning for REST compliance.

5. **Q: What should happen when a server can't satisfy a content negotiation request?**
   **A:** Return 406 Not Acceptable with details about available formats. Include helpful error message, list of supported formats, and documentation links. Don't silently ignore client preferences.

6. **Q: How does content negotiation affect caching strategies?**
   **A:** Responses must include appropriate Vary headers. Caches store separate entries for different Accept combinations. ETags should incorporate content type. Consider cache efficiency when supporting many formats.

7. **Q: What's the difference between server-driven and agent-driven negotiation?**
   **A:** Server-driven: server automatically chooses best representation based on Accept headers. Agent-driven: server returns 300 Multiple Choices with available options, client makes explicit choice. Server-driven is more common and efficient.

8. **Q: How do you implement language negotiation in REST APIs?**
   **A:** Use Accept-Language header from client, match against available languages, consider regional preferences (en-US vs en), provide fallback to default language, and include Content-Language in response with Vary: Accept-Language.

9. **Q: What are the performance implications of content negotiation?**
   **A:** Multiple format support increases server complexity, transformation overhead, cache storage requirements. Mitigate with: format-specific optimizations, efficient serialization, strategic caching, and limiting supported formats to essentials.

10. **Q: How should mobile vs desktop clients handle content negotiation differently?**
    **A:** Mobile clients might prefer: compressed formats (gzip), minimal data (field selection), efficient formats (binary vs text), lower quality images. Desktop clients might prefer: full data, high-quality media, multiple formats for compatibility. Use User-Agent or custom headers to differentiate.
