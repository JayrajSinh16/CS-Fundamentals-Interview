# API Documentation (OpenAPI/Swagger)

Clear documentation accelerates adoption and reduces support burden.

## OpenAPI Basics

- YAML/JSON spec describing endpoints, parameters, schemas
- Tools: Swagger UI, ReDoc, code generators

## Example Skeleton

```yaml
openapi: 3.0.3
info:
  title: Example API
  version: 1.0.0
servers:
  - url: https://api.example.com/v1
paths:
  /users:
    get:
      summary: List users
      parameters:
        - in: query
          name: page
          schema: { type: integer, minimum: 1 }
      responses:
        '200':
          description: OK
          content:
            application/json:
              schema:
                type: object
                properties:
                  data:
                    type: array
                    items: { $ref: '#/components/schemas/User' }
components:
  schemas:
    User:
      type: object
      required: [id, name, email]
      properties:
        id: { type: integer }
        name: { type: string }
        email: { type: string, format: email }
```

## Good Docs Practices

- Examples for requests/responses and errors
- Auth instructions and scopes per endpoint
- Consistent schemas and enums
- Try-it-out consoles and SDKs

## Keeping Docs in Sync

- Contract-first development
- Lint specs (spectral), CI validation
- Generate server stubs and clients

## Interview Questions

1. Contract-first vs code-first—pros/cons?
2. How to document pagination, errors, and auth?
3. How to ensure docs stay current?
4. How to version OpenAPI specs?
5. How to generate SDKs from OpenAPI?
