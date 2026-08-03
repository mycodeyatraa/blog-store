---
title: Error Handling & Status Codes: Testing Negative Edge Cases in Supertest
date: 28-Feb-2026
lastUpdated: 28-Feb-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: ["supertest", "nodejs", "express", "jest", "error-handling", "edge-cases"]
category: API Supertest
categories: ["API Supertest", "Node.js", "Express"]
excerpt: >-
  Build bulletproof REST APIs! Learn how to validate 4xx/5xx HTTP status codes, error payloads, and negative edge cases with Supertest and Jest.
readTime: 8 min read
---

# Error Handling & Status Codes: Testing Negative Edge Cases in Supertest

While happy-path tests confirm that an Express API functions when valid inputs are provided, production systems must gracefully handle bad requests, missing headers, unauthorized tokens, and server exceptions.

**Supertest** makes it seamless to send malformed request payloads, inspect HTTP `4xx` and `5xx` status codes, and validate structured JSON error schemas using **Jest** assertions.

---

## 1. Core Categories of Negative API Edge Cases

- **400 Bad Request**: Invalid JSON schema, missing mandatory fields, invalid data formats.
- **401 Unauthorized / 403 Forbidden**: Missing Bearer token, expired JWT, insufficient role permissions.
- **404 Not Found**: Requesting non-existent API routes or entity resource IDs.
- **422 Unprocessable Entity**: Business validation failures (e.g., duplicate user email).
- **500 Internal Server Error**: Uncaught exception handling and DB connection timeouts.

---

## 2. Implementing Error Handling Tests in Express

Below is a complete Jest / Supertest test suite validating Express error responses:

**tests/error-handling.spec.ts**

```typescript
import request from 'supertest';
import app from '../src/app';
 
describe('Express API Error Handling & Negative Test Suite', () => {
 
  test('should return 400 Bad Request when mandatory email field is missing', async () => {
    const response = await request(app)
      .post('/api/v1/users')
      .send({ name: 'Pankaj Kumar' }) // Missing email
      .set('Accept', 'application/json');
 
    expect(response.status).toBe(400);
    expect(response.body).toHaveProperty('error');
    expect(response.body.error).toBe('Bad Request: Email is required');
  });
 
  test('should return 401 Unauthorized when Bearer token is omitted', async () => {
    const response = await request(app)
      .get('/api/v1/protected/dashboard')
      .set('Accept', 'application/json');
 
    expect(response.status).toBe(401);
    expect(response.body.message).toBe('Missing authorization header');
  });
 
  test('should return 404 Not Found for non-existent endpoint routes', async () => {
    const response = await request(app)
      .get('/api/v1/unknown-endpoint');
 
    expect(response.status).toBe(404);
    expect(response.body.status).toBe('FAIL');
  });
 
  test('should return 422 Unprocessable Entity when registering duplicate email', async () => {
    const payload = { email: 'existing@mycodeyatra.com', password: 'SecretPass123' };
 
    const response = await request(app)
      .post('/api/v1/register')
      .send(payload);
 
    expect(response.status).toBe(422);
    expect(response.body.errors).toContainEqual({
      field: 'email',
      message: 'Email address already registered'
    });
  });
});
```

---

## Key Best Practices

1. **Assert Error Structure**: Validate that error responses return consistent JSON schemas (`{ status, error, timestamp }`).
2. **Prevent Leaking Stack Traces**: Ensure `500` error responses do not leak database connection strings or internal file paths in production.
