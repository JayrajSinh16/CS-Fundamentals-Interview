# Data Validation

Robust validation ensures data quality and prevents security issues. Validate all inputs at the edge and return helpful errors.

## What to Validate

- Types: string, number, boolean, array, object
- Required fields and optional fields with defaults
- Ranges and lengths (min/max)
- Formats: email, UUID, ISO dates, URLs
- Enums and allowed values
- Nested objects and arrays

## Examples

```http
POST /api/products
Content-Type: application/json

{
  "name": "Laptop",
  "price": 1299.99,
  "currency": "USD",
  "category": "electronics",
  "tags": ["work", "portable"],
  "specs": {
    "cpu": "i7",
    "ram_gb": 16,
    "storage_gb": 512
  }
}

201 Created
{
  "id": 1001,
  "name": "Laptop",
  "price": 1299.99,
  "currency": "USD",
  "category": "electronics",
  "tags": ["work", "portable"],
  "specs": { "cpu": "i7", "ram_gb": 16, "storage_gb": 512 }
}
```

Invalid example:
```http
POST /api/products
{
  "name": "",
  "price": -5,
  "currency": "USDX",
  "specs": { "ram_gb": "sixteen" }
}

400 Bad Request
{
  "error": "validation_failed",
  "details": [
    { "field": "name", "code": "required" },
    { "field": "price", "code": "min_value", "min": 0 },
    { "field": "currency", "code": "enum", "allowed": ["USD","EUR","INR"] },
    { "field": "specs.ram_gb", "code": "type_number" }
  ]
}
```

## JSON Schema (example)

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://example.com/schemas/product.schema.json",
  "type": "object",
  "required": ["name", "price", "currency"],
  "properties": {
    "name": { "type": "string", "minLength": 1 },
    "price": { "type": "number", "minimum": 0 },
    "currency": { "type": "string", "enum": ["USD", "EUR", "INR"] },
    "category": { "type": "string" },
    "tags": { "type": "array", "items": { "type": "string" }, "maxItems": 10 },
    "specs": {
      "type": "object",
      "properties": {
        "cpu": { "type": "string" },
        "ram_gb": { "type": "integer", "minimum": 1 },
        "storage_gb": { "type": "integer", "minimum": 16 }
      },
      "additionalProperties": false
    }
  },
  "additionalProperties": false
}
```

## Best Practices

- Validate at the edge (controller/middleware)
- Use schemas for consistency and docs
- Return all validation errors together
- Sanitize inputs to prevent XSS/SQLi
- Enforce limits (payload size, array length)

## Interview Questions

1. How do you structure validation errors for UX?
2. JSON Schema vs custom validators trade-offs?
3. Where should validation live in layered architecture?
4. How do you validate nested arrays/objects?
5. How do you keep validation and OpenAPI specs in sync?
