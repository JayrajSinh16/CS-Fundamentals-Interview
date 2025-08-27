# Mock APIs

Mock APIs simulate endpoints for development, testing, and onboarding.

## Benefits

- Parallel development (frontend/backend)
- Consistent, predictable responses
- Test edge cases and error conditions

## Approaches

- Static mocks (JSON files, mock servers)
- Contract-driven mocks (OpenAPI + Prism/MSW/Pact)
- Dynamic mocks with scripts (Node/Express, WireMock)

## Example (Prism)

```bash
prism mock openapi.yaml
```

## Example (Express)

```js
const express = require('express');
const app = express();
app.get('/api/users/:id', (req, res) => {
  res.json({ id: req.params.id, name: 'Mock User' });
});
app.listen(3001);
```

## Best Practices

- Base mocks on OpenAPI schemas
- Support scenario switching (happy path, errors)
- Record/replay for integration testing

## Interview Questions

1. How to keep mocks in sync with API contracts?
2. When to use static vs dynamic mocks?
3. How to simulate rate limiting, delays, and errors?
4. How do mocks help with CI/CD pipelines?
5. How to transition from mocks to real services?
