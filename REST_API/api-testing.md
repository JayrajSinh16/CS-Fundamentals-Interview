# Testing REST APIs

Comprehensive testing is essential for building reliable and maintainable REST APIs. Testing ensures that APIs work correctly, handle edge cases gracefully, and maintain backward compatibility. This guide covers various testing strategies, tools, and best practices for REST API testing.

## Types of API Testing

### 1. Unit Testing

Test individual API components in isolation.

```http
# Test endpoint: POST /api/users
# Unit test for user creation logic

// JavaScript/Jest example
describe('User Creation API', () => {
  test('should create user with valid data', async () => {
    const validUserData = {
      name: 'John Doe',
      email: 'john@example.com',
      age: 30
    };

    const response = await request(app)
      .post('/api/users')
      .send(validUserData)
      .expect(201);

    expect(response.body).toMatchObject({
      id: expect.any(Number),
      name: 'John Doe',
      email: 'john@example.com',
      age: 30,
      created_at: expect.any(String)
    });

    expect(response.headers.location).toBe(`/api/users/${response.body.id}`);
  });

  test('should reject invalid email format', async () => {
    const invalidUserData = {
      name: 'John Doe',
      email: 'invalid-email',
      age: 30
    };

    const response = await request(app)
      .post('/api/users')
      .send(invalidUserData)
      .expect(400);

    expect(response.body).toMatchObject({
      error: 'validation_failed',
      details: expect.arrayContaining([
        expect.objectContaining({
          field: 'email',
          code: 'invalid_format'
        })
      ])
    });
  });

  test('should handle duplicate email', async () => {
    // Setup: Create user first
    await User.create({
      name: 'Existing User',
      email: 'existing@example.com'
    });

    const duplicateUserData = {
      name: 'New User',
      email: 'existing@example.com',
      age: 25
    };

    const response = await request(app)
      .post('/api/users')
      .send(duplicateUserData)
      .expect(409);

    expect(response.body).toMatchObject({
      error: 'resource_conflict',
      conflicting_field: 'email'
    });
  });
});
```

### 2. Integration Testing

Test API interactions with external systems and databases.

```http
# Integration test with database
describe('User API Integration', () => {
  beforeEach(async () => {
    // Clean database before each test
    await User.deleteMany({});
  });

  test('should persist user to database', async () => {
    const userData = {
      name: 'John Doe',
      email: 'john@example.com'
    };

    const response = await request(app)
      .post('/api/users')
      .send(userData)
      .expect(201);

    // Verify data persisted in database
    const savedUser = await User.findById(response.body.id);
    expect(savedUser).toBeTruthy();
    expect(savedUser.name).toBe('John Doe');
    expect(savedUser.email).toBe('john@example.com');
  });

  test('should handle database connection failure', async () => {
    // Simulate database failure
    await mongoose.connection.close();

    const userData = {
      name: 'John Doe',
      email: 'john@example.com'
    };

    const response = await request(app)
      .post('/api/users')
      .send(userData)
      .expect(503);

    expect(response.body).toMatchObject({
      error: 'service_unavailable',
      message: expect.stringContaining('database')
    });
  });
});

# Integration test with external API
describe('External Payment Service Integration', () => {
  test('should process payment successfully', async () => {
    // Mock external service
    nock('https://payment-service.com')
      .post('/api/charges')
      .reply(200, {
        id: 'charge_123',
        status: 'succeeded',
        amount: 2000
      });

    const paymentData = {
      amount: 2000,
      currency: 'usd',
      source: 'tok_visa'
    };

    const response = await request(app)
      .post('/api/payments')
      .send(paymentData)
      .expect(201);

    expect(response.body).toMatchObject({
      id: expect.any(String),
      status: 'completed',
      external_charge_id: 'charge_123'
    });
  });

  test('should handle external service timeout', async () => {
    // Mock service timeout
    nock('https://payment-service.com')
      .post('/api/charges')
      .delay(5000)
      .reply(200, {});

    const paymentData = {
      amount: 2000,
      currency: 'usd',
      source: 'tok_visa'
    };

    const response = await request(app)
      .post('/api/payments')
      .send(paymentData)
      .expect(504);

    expect(response.body).toMatchObject({
      error: 'gateway_timeout',
      upstream_service: 'payment-service'
    });
  });
});
```

### 3. Contract Testing

Ensure API contracts are maintained between services.

```http
# Pact consumer test (JavaScript)
describe('User API Consumer Contract', () => {
  const provider = new Pact({
    consumer: 'UserApp',
    provider: 'UserService',
    port: 1234
  });

  beforeAll(() => provider.setup());
  afterAll(() => provider.finalize());

  test('should get user by ID', async () => {
    await provider.addInteraction({
      state: 'user 123 exists',
      uponReceiving: 'a request for user 123',
      withRequest: {
        method: 'GET',
        path: '/api/users/123',
        headers: {
          'Authorization': 'Bearer valid_token'
        }
      },
      willRespondWith: {
        status: 200,
        headers: {
          'Content-Type': 'application/json'
        },
        body: {
          id: 123,
          name: 'John Doe',
          email: 'john@example.com',
          created_at: Matchers.iso8601DateTime()
        }
      }
    });

    const client = new UserApiClient('http://localhost:1234');
    const user = await client.getUser(123);

    expect(user.id).toBe(123);
    expect(user.name).toBe('John Doe');
  });

  test('should handle user not found', async () => {
    await provider.addInteraction({
      state: 'user 999 does not exist',
      uponReceiving: 'a request for user 999',
      withRequest: {
        method: 'GET',
        path: '/api/users/999'
      },
      willRespondWith: {
        status: 404,
        headers: {
          'Content-Type': 'application/json'
        },
        body: {
          error: 'resource_not_found',
          message: 'User with ID 999 not found'
        }
      }
    });

    const client = new UserApiClient('http://localhost:1234');
    
    await expect(client.getUser(999))
      .rejects
      .toThrow('User with ID 999 not found');
  });
});

# OpenAPI contract validation
describe('OpenAPI Contract Validation', () => {
  const openApiSpec = require('../docs/openapi.json');

  test('should validate response against OpenAPI schema', async () => {
    const response = await request(app)
      .get('/api/users/123')
      .expect(200);

    // Validate response against OpenAPI schema
    const validator = new OpenAPIResponseValidator({
      openApiSpec,
      path: '/users/{id}',
      method: 'get',
      statusCode: 200
    });

    const validationErrors = validator.validateResponse(response.body);
    expect(validationErrors).toBeUndefined();
  });

  test('should validate request against OpenAPI schema', async () => {
    const requestBody = {
      name: 'John Doe',
      email: 'john@example.com',
      age: 30
    };

    const validator = new OpenAPIRequestValidator({
      openApiSpec,
      path: '/users',
      method: 'post'
    });

    const validationErrors = validator.validateRequest({
      body: requestBody,
      headers: { 'content-type': 'application/json' }
    });

    expect(validationErrors).toBeUndefined();
  });
});
```

### 4. End-to-End Testing

Test complete user workflows across the entire API.

```http
# E2E test for user registration and login flow
describe('User Registration and Login E2E', () => {
  test('complete user journey', async () => {
    // Step 1: Register new user
    const registrationData = {
      name: 'Jane Smith',
      email: 'jane.smith@example.com',
      password: 'SecurePassword123!'
    };

    const registrationResponse = await request(app)
      .post('/api/auth/register')
      .send(registrationData)
      .expect(201);

    expect(registrationResponse.body).toMatchObject({
      id: expect.any(Number),
      name: 'Jane Smith',
      email: 'jane.smith@example.com'
    });

    const userId = registrationResponse.body.id;

    // Step 2: Verify email (simulate email verification)
    const verificationToken = 'generated_verification_token';
    await request(app)
      .post('/api/auth/verify-email')
      .send({ token: verificationToken })
      .expect(200);

    // Step 3: Login with credentials
    const loginData = {
      email: 'jane.smith@example.com',
      password: 'SecurePassword123!'
    };

    const loginResponse = await request(app)
      .post('/api/auth/login')
      .send(loginData)
      .expect(200);

    expect(loginResponse.body).toMatchObject({
      access_token: expect.any(String),
      token_type: 'Bearer',
      expires_in: expect.any(Number)
    });

    const accessToken = loginResponse.body.access_token;

    // Step 4: Access protected resource
    const profileResponse = await request(app)
      .get('/api/users/profile')
      .set('Authorization', `Bearer ${accessToken}`)
      .expect(200);

    expect(profileResponse.body).toMatchObject({
      id: userId,
      name: 'Jane Smith',
      email: 'jane.smith@example.com',
      email_verified: true
    });

    // Step 5: Update profile
    const updateData = {
      name: 'Jane Smith-Johnson',
      bio: 'Software developer'
    };

    const updateResponse = await request(app)
      .patch('/api/users/profile')
      .set('Authorization', `Bearer ${accessToken}`)
      .send(updateData)
      .expect(200);

    expect(updateResponse.body.name).toBe('Jane Smith-Johnson');
    expect(updateResponse.body.bio).toBe('Software developer');

    // Step 6: Logout
    await request(app)
      .post('/api/auth/logout')
      .set('Authorization', `Bearer ${accessToken}`)
      .expect(204);

    // Step 7: Verify token is invalidated
    await request(app)
      .get('/api/users/profile')
      .set('Authorization', `Bearer ${accessToken}`)
      .expect(401);
  });
});

# E2E test for e-commerce order flow
describe('E-commerce Order Flow E2E', () => {
  let authToken;
  let customerId;

  beforeAll(async () => {
    // Setup authenticated customer
    const loginResponse = await request(app)
      .post('/api/auth/login')
      .send({
        email: 'customer@example.com',
        password: 'password'
      });
    
    authToken = loginResponse.body.access_token;
    customerId = loginResponse.body.user.id;
  });

  test('complete order workflow', async () => {
    // Step 1: Browse products
    const productsResponse = await request(app)
      .get('/api/products?category=electronics&limit=10')
      .expect(200);

    expect(productsResponse.body.products).toHaveLength(10);
    const selectedProduct = productsResponse.body.products[0];

    // Step 2: Add to cart
    const cartResponse = await request(app)
      .post('/api/cart/items')
      .set('Authorization', `Bearer ${authToken}`)
      .send({
        product_id: selectedProduct.id,
        quantity: 2
      })
      .expect(201);

    expect(cartResponse.body.total_items).toBe(2);

    // Step 3: Update cart quantity
    const cartItemId = cartResponse.body.item_id;
    await request(app)
      .patch(`/api/cart/items/${cartItemId}`)
      .set('Authorization', `Bearer ${authToken}`)
      .send({ quantity: 3 })
      .expect(200);

    // Step 4: Get updated cart
    const updatedCartResponse = await request(app)
      .get('/api/cart')
      .set('Authorization', `Bearer ${authToken}`)
      .expect(200);

    expect(updatedCartResponse.body.total_items).toBe(3);
    const totalAmount = updatedCartResponse.body.total_amount;

    // Step 5: Create order
    const orderData = {
      shipping_address: {
        street: '123 Main St',
        city: 'Anytown',
        state: 'NY',
        zip: '12345',
        country: 'US'
      },
      payment_method: {
        type: 'credit_card',
        token: 'payment_token_123'
      }
    };

    const orderResponse = await request(app)
      .post('/api/orders')
      .set('Authorization', `Bearer ${authToken}`)
      .send(orderData)
      .expect(201);

    expect(orderResponse.body).toMatchObject({
      id: expect.any(String),
      customer_id: customerId,
      total_amount: totalAmount,
      status: 'pending',
      items: expect.arrayContaining([
        expect.objectContaining({
          product_id: selectedProduct.id,
          quantity: 3
        })
      ])
    });

    const orderId = orderResponse.body.id;

    // Step 6: Process payment
    const paymentResponse = await request(app)
      .post(`/api/orders/${orderId}/payment`)
      .set('Authorization', `Bearer ${authToken}`)
      .expect(200);

    expect(paymentResponse.body.status).toBe('payment_processing');

    // Step 7: Check order status
    const orderStatusResponse = await request(app)
      .get(`/api/orders/${orderId}`)
      .set('Authorization', `Bearer ${authToken}`)
      .expect(200);

    expect(orderStatusResponse.body.status).toMatch(/^(paid|processing)$/);

    // Step 8: Verify cart is cleared
    const finalCartResponse = await request(app)
      .get('/api/cart')
      .set('Authorization', `Bearer ${authToken}`)
      .expect(200);

    expect(finalCartResponse.body.total_items).toBe(0);
  });
});
```

## Performance Testing

### 1. Load Testing

Test API performance under expected load conditions.

```http
# Artillery.js load test configuration
# artillery-config.yml
config:
  target: 'https://api.example.com'
  phases:
    - duration: 60
      arrivalRate: 10  # 10 requests per second
      name: "Warm up"
    - duration: 120
      arrivalRate: 50  # 50 requests per second
      name: "Sustained load"
    - duration: 60
      arrivalRate: 100 # 100 requests per second
      name: "Peak load"

scenarios:
  - name: "Get users list"
    weight: 40
    flow:
      - get:
          url: "/api/users"
          headers:
            Authorization: "Bearer {{ auth_token }}"
          capture:
            - json: "$.users[0].id"
              as: "user_id"
      - think: 2
      - get:
          url: "/api/users/{{ user_id }}"
          headers:
            Authorization: "Bearer {{ auth_token }}"

  - name: "Create and update user"
    weight: 30
    flow:
      - post:
          url: "/api/users"
          headers:
            Authorization: "Bearer {{ auth_token }}"
            Content-Type: "application/json"
          json:
            name: "Load Test User {{ $randomString() }}"
            email: "user{{ $randomInt(1, 10000) }}@example.com"
          capture:
            - json: "$.id"
              as: "new_user_id"
      - think: 1
      - patch:
          url: "/api/users/{{ new_user_id }}"
          headers:
            Authorization: "Bearer {{ auth_token }}"
            Content-Type: "application/json"
          json:
            bio: "Updated during load test"

  - name: "Search operations"
    weight: 30
    flow:
      - get:
          url: "/api/users?search={{ $randomString() }}&limit=20"
          headers:
            Authorization: "Bearer {{ auth_token }}"
      - think: 1
      - get:
          url: "/api/users?sort=created_at&order=desc&limit=50"
          headers:
            Authorization: "Bearer {{ auth_token }}"

# Run load test
# artillery run artillery-config.yml

# Expected output metrics:
# http.codes.200: 14500
# http.codes.201: 180
# http.codes.404: 25
# http.response_time.min: 45
# http.response_time.max: 2340
# http.response_time.median: 125
# http.response_time.p95: 580
# http.response_time.p99: 890
```

### 2. Stress Testing

Test API behavior under extreme load conditions.

```http
# K6 stress test script
import http from 'k6/http';
import { check, sleep } from 'k6';
import { Rate } from 'k6/metrics';

const errorRate = new Rate('errors');

export let options = {
  stages: [
    { duration: '2m', target: 100 },   // Ramp up to 100 users
    { duration: '5m', target: 100 },   // Stay at 100 users
    { duration: '2m', target: 200 },   // Ramp up to 200 users  
    { duration: '5m', target: 200 },   // Stay at 200 users
    { duration: '2m', target: 300 },   // Ramp up to 300 users
    { duration: '5m', target: 300 },   // Stay at 300 users
    { duration: '2m', target: 0 },     // Ramp down to 0 users
  ],
  thresholds: {
    'http_req_duration': ['p(95)<500'], // 95% of requests under 500ms
    'http_req_failed': ['rate<0.05'],   // Less than 5% errors
    'errors': ['rate<0.1']              // Less than 10% application errors
  }
};

const BASE_URL = 'https://api.example.com';
const AUTH_TOKEN = 'your_test_token';

export default function() {
  const headers = {
    'Authorization': `Bearer ${AUTH_TOKEN}`,
    'Content-Type': 'application/json'
  };

  // Test 1: Get users list (read-heavy operation)
  let response = http.get(`${BASE_URL}/api/users?limit=20`, {
    headers: { 'Authorization': `Bearer ${AUTH_TOKEN}` }
  });

  let result = check(response, {
    'users list status is 200': (r) => r.status === 200,
    'users list response time < 200ms': (r) => r.timings.duration < 200,
    'users list has data': (r) => {
      try {
        const data = JSON.parse(r.body);
        return data.users && Array.isArray(data.users);
      } catch (e) {
        return false;
      }
    }
  });
  
  errorRate.add(!result);

  sleep(1);

  // Test 2: Create user (write operation)
  const userData = {
    name: `Stress Test User ${Math.random()}`,
    email: `stress${Math.floor(Math.random() * 10000)}@example.com`,
    age: Math.floor(Math.random() * 50) + 18
  };

  response = http.post(`${BASE_URL}/api/users`, JSON.stringify(userData), {
    headers: headers
  });

  result = check(response, {
    'user creation status is 201': (r) => r.status === 201,
    'user creation response time < 500ms': (r) => r.timings.duration < 500,
    'user created successfully': (r) => {
      try {
        const data = JSON.parse(r.body);
        return data.id && data.name === userData.name;
      } catch (e) {
        return false;
      }
    }
  });

  errorRate.add(!result);

  if (response.status === 201) {
    const createdUser = JSON.parse(response.body);
    
    // Test 3: Update created user
    const updateData = {
      bio: `Updated during stress test at ${new Date().toISOString()}`
    };

    response = http.patch(`${BASE_URL}/api/users/${createdUser.id}`, 
      JSON.stringify(updateData), { headers: headers });

    result = check(response, {
      'user update status is 200': (r) => r.status === 200,
      'user update response time < 300ms': (r) => r.timings.duration < 300
    });

    errorRate.add(!result);
  }

  sleep(Math.random() * 2);
}

export function handleSummary(data) {
  return {
    'stress-test-results.json': JSON.stringify(data, null, 2),
    stdout: textSummary(data, { indent: ' ', enableColors: true })
  };
}
```

### 3. Spike Testing

Test API response to sudden traffic spikes.

```http
# Spike test configuration
# spike-test.js for K6
import http from 'k6/http';
import { check } from 'k6';

export let options = {
  stages: [
    { duration: '1m', target: 20 },    // Normal load
    { duration: '10s', target: 200 },  // Sudden spike
    { duration: '2m', target: 200 },   // Maintain spike
    { duration: '10s', target: 20 },   // Drop back to normal
    { duration: '1m', target: 20 },    // Continue normal load
  ],
  thresholds: {
    'http_req_duration': ['p(95)<1000'], // Allow higher latency during spikes
    'http_req_failed': ['rate<0.1'],     // Accept some failures during spike
  }
};

export default function() {
  const response = http.get('https://api.example.com/api/users');
  
  check(response, {
    'status is 200 or 503': (r) => r.status === 200 || r.status === 503,
    'response time acceptable': (r) => r.timings.duration < 2000,
  });

  // During spike, some requests may be rate limited
  if (response.status === 429) {
    console.log('Rate limited - this is expected during spike');
  }
}
```

## Security Testing

### 1. Authentication and Authorization Testing

Test security controls and access restrictions.

```http
# Authentication testing
describe('Authentication Security', () => {
  test('should reject requests without authentication', async () => {
    const response = await request(app)
      .get('/api/protected-resource')
      .expect(401);

    expect(response.body).toMatchObject({
      error: 'authentication_required'
    });
  });

  test('should reject malformed tokens', async () => {
    const response = await request(app)
      .get('/api/protected-resource')
      .set('Authorization', 'Bearer malformed.token.here')
      .expect(401);

    expect(response.body).toMatchObject({
      error: 'invalid_token'
    });
  });

  test('should reject expired tokens', async () => {
    const expiredToken = jwt.sign(
      { user_id: 123, exp: Math.floor(Date.now() / 1000) - 3600 },
      process.env.JWT_SECRET
    );

    const response = await request(app)
      .get('/api/protected-resource')
      .set('Authorization', `Bearer ${expiredToken}`)
      .expect(401);

    expect(response.body).toMatchObject({
      error: 'token_expired'
    });
  });

  test('should enforce role-based access control', async () => {
    const userToken = createTokenForUser({ id: 123, role: 'user' });
    
    const response = await request(app)
      .get('/api/admin/sensitive-data')
      .set('Authorization', `Bearer ${userToken}`)
      .expect(403);

    expect(response.body).toMatchObject({
      error: 'insufficient_permissions',
      required_role: 'admin'
    });
  });
});

# Input validation security testing
describe('Input Validation Security', () => {
  test('should prevent SQL injection', async () => {
    const maliciousInput = {
      search: "'; DROP TABLE users; --"
    };

    const response = await request(app)
      .get('/api/users')
      .query(maliciousInput)
      .expect(400);

    expect(response.body).toMatchObject({
      error: 'invalid_input',
      message: expect.stringContaining('dangerous')
    });
  });

  test('should prevent XSS in response data', async () => {
    const xssPayload = {
      bio: '<script>alert("xss")</script>'
    };

    const response = await request(app)
      .post('/api/users')
      .send({
        name: 'Test User',
        email: 'test@example.com',
        bio: xssPayload.bio
      })
      .expect(201);

    // Bio should be escaped/sanitized in response
    expect(response.body.bio).not.toContain('<script>');
    expect(response.body.bio).toContain('&lt;script&gt;');
  });

  test('should enforce request size limits', async () => {
    const largePayload = {
      data: 'x'.repeat(10 * 1024 * 1024) // 10MB string
    };

    const response = await request(app)
      .post('/api/data')
      .send(largePayload)
      .expect(413);

    expect(response.body).toMatchObject({
      error: 'payload_too_large'
    });
  });

  test('should validate file upload types', async () => {
    const response = await request(app)
      .post('/api/upload')
      .attach('file', Buffer.from('malicious content'), 'malware.exe')
      .expect(400);

    expect(response.body).toMatchObject({
      error: 'invalid_file_type',
      allowed_types: expect.arrayContaining(['image/jpeg', 'image/png'])
    });
  });
});

# Rate limiting security testing
describe('Rate Limiting Security', () => {
  test('should enforce rate limits', async () => {
    const requests = [];
    
    // Make requests up to rate limit
    for (let i = 0; i < 15; i++) {
      requests.push(
        request(app)
          .get('/api/data')
          .set('X-API-Key', 'test_key')
      );
    }

    const responses = await Promise.all(requests);
    
    // Some requests should be rate limited
    const rateLimitedResponses = responses.filter(r => r.status === 429);
    expect(rateLimitedResponses.length).toBeGreaterThan(0);

    const rateLimitedResponse = rateLimitedResponses[0];
    expect(rateLimitedResponse.body).toMatchObject({
      error: 'rate_limit_exceeded'
    });
    expect(rateLimitedResponse.headers['retry-after']).toBeDefined();
  });

  test('should handle distributed rate limiting', async () => {
    // Test rate limiting across multiple API keys/users
    const apiKey1 = 'test_key_1';
    const apiKey2 = 'test_key_2';

    // Exhaust rate limit for key1
    for (let i = 0; i < 10; i++) {
      await request(app)
        .get('/api/data')
        .set('X-API-Key', apiKey1);
    }

    // Key1 should be rate limited
    const key1Response = await request(app)
      .get('/api/data')
      .set('X-API-Key', apiKey1)
      .expect(429);

    // Key2 should still work
    const key2Response = await request(app)
      .get('/api/data')
      .set('X-API-Key', apiKey2)
      .expect(200);
  });
});
```

### 2. Data Security Testing

Test data protection and privacy controls.

```http
# Data masking and privacy testing
describe('Data Security', () => {
  test('should mask sensitive data in responses', async () => {
    const user = await User.create({
      name: 'John Doe',
      email: 'john@example.com',
      ssn: '123-45-6789',
      credit_card: '4111-1111-1111-1111'
    });

    const response = await request(app)
      .get(`/api/users/${user.id}`)
      .set('Authorization', 'Bearer regular_user_token')
      .expect(200);

    expect(response.body.ssn).toMatch(/\*\*\*-\*\*-\d{4}/);
    expect(response.body.credit_card).toMatch(/\*\*\*\*-\*\*\*\*-\*\*\*\*-\d{4}/);
  });

  test('should enforce data access permissions', async () => {
    const user1 = await User.create({
      name: 'User 1',
      email: 'user1@example.com'
    });

    const user2Token = createTokenForUser({ id: 456, role: 'user' });

    const response = await request(app)
      .get(`/api/users/${user1.id}/private-data`)
      .set('Authorization', `Bearer ${user2Token}`)
      .expect(403);

    expect(response.body).toMatchObject({
      error: 'data_access_denied',
      message: 'You can only access your own private data'
    });
  });

  test('should audit data access', async () => {
    const adminToken = createTokenForUser({ id: 789, role: 'admin' });

    await request(app)
      .get('/api/users/123/sensitive-data')
      .set('Authorization', `Bearer ${adminToken}`)
      .expect(200);

    // Verify audit log entry was created
    const auditLog = await AuditLog.findOne({
      user_id: 789,
      action: 'data_access',
      resource: 'user_sensitive_data',
      resource_id: '123'
    });

    expect(auditLog).toBeTruthy();
    expect(auditLog.timestamp).toBeDefined();
  });
});
```

## Test Automation and CI/CD

### 1. Test Pipeline Configuration

Automate testing in continuous integration.

```yaml
# GitHub Actions workflow (.github/workflows/api-tests.yml)
name: API Tests

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  unit-tests:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:13
        env:
          POSTGRES_PASSWORD: postgres
          POSTGRES_DB: api_test
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 5432:5432

    steps:
    - uses: actions/checkout@v3
    
    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '18'
        cache: 'npm'

    - name: Install dependencies
      run: npm ci

    - name: Run unit tests
      run: npm run test:unit
      env:
        DATABASE_URL: postgres://postgres:postgres@localhost:5432/api_test
        JWT_SECRET: test_secret
        NODE_ENV: test

    - name: Upload coverage reports
      uses: codecov/codecov-action@v3
      with:
        file: ./coverage/lcov.info

  integration-tests:
    runs-on: ubuntu-latest
    needs: unit-tests
    services:
      postgres:
        image: postgres:13
        env:
          POSTGRES_PASSWORD: postgres
          POSTGRES_DB: api_test
      redis:
        image: redis:6
        options: >-
          --health-cmd "redis-cli ping"
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

    steps:
    - uses: actions/checkout@v3
    
    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '18'
        cache: 'npm'

    - name: Install dependencies
      run: npm ci

    - name: Run database migrations
      run: npm run db:migrate
      env:
        DATABASE_URL: postgres://postgres:postgres@localhost:5432/api_test

    - name: Run integration tests
      run: npm run test:integration
      env:
        DATABASE_URL: postgres://postgres:postgres@localhost:5432/api_test
        REDIS_URL: redis://localhost:6379
        JWT_SECRET: test_secret

  contract-tests:
    runs-on: ubuntu-latest
    needs: unit-tests
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '18'
        cache: 'npm'

    - name: Install dependencies
      run: npm ci

    - name: Run Pact consumer tests
      run: npm run test:pact:consumer

    - name: Publish Pact contracts
      run: npm run pact:publish
      env:
        PACT_BROKER_BASE_URL: ${{ secrets.PACT_BROKER_URL }}
        PACT_BROKER_TOKEN: ${{ secrets.PACT_BROKER_TOKEN }}

  performance-tests:
    runs-on: ubuntu-latest
    if: github.event_name == 'push' && github.ref == 'refs/heads/main'
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Setup K6
      run: |
        sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys C5AD17C747E3415A3642D57D77C6C491D6AC1D69
        echo "deb https://dl.k6.io/deb stable main" | sudo tee /etc/apt/sources.list.d/k6.list
        sudo apt-get update
        sudo apt-get install k6

    - name: Run performance tests
      run: k6 run tests/performance/load-test.js
      env:
        API_BASE_URL: ${{ secrets.STAGING_API_URL }}
        API_TOKEN: ${{ secrets.STAGING_API_TOKEN }}

    - name: Upload performance results
      uses: actions/upload-artifact@v3
      with:
        name: performance-results
        path: performance-results.json

  security-tests:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '18'
        cache: 'npm'

    - name: Install dependencies
      run: npm ci

    - name: Run security tests
      run: npm run test:security

    - name: Run OWASP ZAP scan
      uses: zaproxy/action-baseline@v0.7.0
      with:
        target: 'http://localhost:3000'
        docker_name: 'owasp/zap2docker-stable'
        rules_file_name: '.zap/rules.tsv'
        cmd_options: '-a'
```

### 2. Test Data Management

Manage test data consistently across test environments.

```javascript
// Test data factory (testDataFactory.js)
class TestDataFactory {
  static async createUser(overrides = {}) {
    const defaultData = {
      name: 'Test User',
      email: `test${Date.now()}@example.com`,
      age: 30,
      role: 'user',
      status: 'active'
    };

    const userData = { ...defaultData, ...overrides };
    return await User.create(userData);
  }

  static async createProduct(overrides = {}) {
    const defaultData = {
      name: 'Test Product',
      description: 'A test product',
      price: 99.99,
      category: 'electronics',
      stock: 100
    };

    const productData = { ...defaultData, ...overrides };
    return await Product.create(productData);
  }

  static async createOrder(userId, items = [], overrides = {}) {
    const defaultData = {
      customer_id: userId,
      status: 'pending',
      total_amount: 0,
      items: items
    };

    const orderData = { ...defaultData, ...overrides };
    orderData.total_amount = items.reduce((sum, item) => 
      sum + (item.price * item.quantity), 0);

    return await Order.create(orderData);
  }

  static generateApiKey(prefix = 'test') {
    return `${prefix}_${Math.random().toString(36).substring(2, 15)}`;
  }

  static generateJwtToken(payload = {}) {
    const defaultPayload = {
      user_id: 123,
      role: 'user',
      exp: Math.floor(Date.now() / 1000) + 3600 // 1 hour
    };

    return jwt.sign({ ...defaultPayload, ...payload }, process.env.JWT_SECRET);
  }
}

// Test database setup (testSetup.js)
class TestDatabaseManager {
  static async setup() {
    // Create test database if it doesn't exist
    await this.createTestDatabase();
    
    // Run migrations
    await this.runMigrations();
    
    // Seed minimal required data
    await this.seedRequiredData();
  }

  static async cleanup() {
    // Clear all test data but keep schema
    await this.clearAllTables();
  }

  static async clearAllTables() {
    const tables = ['orders', 'products', 'users', 'api_keys'];
    
    for (const table of tables) {
      await db.query(`TRUNCATE TABLE ${table} RESTART IDENTITY CASCADE`);
    }
  }

  static async seedRequiredData() {
    // Create admin user for tests
    await TestDataFactory.createUser({
      email: 'admin@test.com',
      role: 'admin'
    });

    // Create test API key
    await ApiKey.create({
      key: TestDataFactory.generateApiKey('test'),
      name: 'Test API Key',
      permissions: ['read', 'write']
    });
  }
}

// Global test setup
beforeAll(async () => {
  await TestDatabaseManager.setup();
});

beforeEach(async () => {
  await TestDatabaseManager.cleanup();
  await TestDatabaseManager.seedRequiredData();
});

afterAll(async () => {
  await db.close();
});
```

## Interview Questions

1. **Q: What are the different types of API testing and when would you use each?**
   **A:** Unit testing for individual components, integration testing for service interactions, contract testing for API agreements, end-to-end testing for complete workflows, performance testing for load/stress scenarios, and security testing for vulnerability assessment. Choose based on what aspect of the API you need to validate.

2. **Q: How do you test authentication and authorization in REST APIs?**
   **A:** Test missing/invalid tokens (401), expired tokens (401), insufficient permissions (403), role-based access control, token lifecycle, session management, and security boundaries between users/resources. Use test users with different roles and validate access controls.

3. **Q: What strategies help ensure API contract compatibility during testing?**
   **A:** Use contract testing frameworks (Pact), OpenAPI schema validation, version compatibility tests, backward compatibility checks, consumer-driven contracts, and automated contract verification in CI/CD pipelines.

4. **Q: How do you implement effective performance testing for APIs?**
   **A:** Use load testing tools (K6, Artillery), test different load patterns (sustained, spike, soak), monitor response times and error rates, test with realistic data volumes, validate rate limiting, and establish performance baselines with SLA thresholds.

5. **Q: What are the best practices for test data management in API testing?**
   **A:** Use test data factories for consistent data creation, implement database cleanup between tests, use separate test databases, create minimal seed data, avoid hard-coded test data, and implement data isolation between test scenarios.

6. **Q: How do you test error handling and edge cases in APIs?**
   **A:** Test all error status codes, validate error response formats, test boundary conditions, simulate external service failures, test malformed requests, verify timeout handling, and test error propagation through system layers.

7. **Q: What security testing approaches are essential for REST APIs?**
   **A:** Test authentication bypass, authorization flaws, input validation (XSS, SQL injection), rate limiting effectiveness, data exposure, HTTPS enforcement, CORS configuration, and implement OWASP API Security testing guidelines.

8. **Q: How do you integrate API testing into CI/CD pipelines effectively?**
   **A:** Separate test types by execution time (unit → integration → e2e), use test containers for dependencies, implement parallel test execution, generate test reports, fail builds on test failures, and use different test suites for different pipeline stages.

9. **Q: What approaches help test APIs with external dependencies?**
   **A:** Use mocking/stubbing for external services, implement contract testing with providers, create test doubles, use service virtualization, test circuit breaker patterns, and validate fallback behaviors when dependencies fail.

10. **Q: How do you measure and improve API test coverage?**
    **A:** Track code coverage, endpoint coverage, error path coverage, use coverage tools (Istanbul, NYC), analyze untested scenarios, implement coverage gates in CI/CD, focus on critical business paths, and balance coverage with test maintainability.
