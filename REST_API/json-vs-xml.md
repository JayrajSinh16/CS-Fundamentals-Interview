# JSON vs XML

JSON and XML are common data formats for APIs. JSON has become the default for web APIs, while XML still appears in enterprise or legacy systems.

## JSON

- Lightweight, human-readable, native in JavaScript
- Data types: object, array, string, number, boolean, null
- Example:
```json
{
  "id": 123,
  "name": "John",
  "roles": ["user", "admin"],
  "profile": { "bio": "Hello" }
}
```

Pros:
- Smaller payloads
- Easy to parse and use in JS/TS
- Great tooling, schema via JSON Schema

Cons:
- No comments, less verbose metadata
- Limited support for attributes/namespace concepts (compared to XML)

## XML

- Markup language with tags, attributes, and namespaces
- Example:
```xml
<user id="123">
  <name>John</name>
  <roles>
    <role>user</role>
    <role>admin</role>
  </roles>
  <profile>
    <bio>Hello</bio>
  </profile>
</user>
```

Pros:
- Rich metadata, attributes, and validation (XSD)
- Namespaces, transformations (XSLT)

Cons:
- Verbose payloads
- Heavier parsing

## When to Use Which

- JSON: Web/mobile clients, modern APIs, microservices
- XML: Enterprise systems, document-centric data, SOAP services

## Coexistence

- Content negotiation: support both via Accept/Content-Type headers
- Provide schemas: JSON Schema for JSON, XSD for XML

## Interview Questions

1. Why is JSON preferred for most web APIs?
2. How do you validate JSON vs XML payloads?
3. When would you still choose XML today?
4. How to implement content negotiation to serve JSON and XML?
5. What are pros/cons of JSON vs XML in performance and tooling?
