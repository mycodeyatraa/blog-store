---
title: Authentication & Authorization
date: 24-Feb-2025
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [supertest, authentication, jwt, basic-auth, bearer, jest]
category: API Testing
categories: [API Testing, NodeJS, Security, TypeScript]
excerpt: >-
  Learn how to bypass security gateways in your test automation suite by mastering Basic Auth and dynamic JWT Bearer Tokens in SuperTest.
readTime: 5 min read
---

In the modern web, almost all APIs are secured behind some form of Authentication. The days of simply firing GET requests to public endpoints are long gone. 

If you want to build a real-world automation framework, you must know how to negotiate with Authentication gateways. In this tutorial, we will explore how to handle the two most common authentication mechanisms in **SuperTest**: Basic Auth and JWT Bearer Tokens.

---

## 1. Basic Authentication

Basic Authentication is a legacy (but still widely used) mechanism where a client sends HTTP requests with an `Authorization` header that contains the word Basic followed by a base64-encoded string of the username and password.

SuperTest makes this incredibly easy with its built-in `.auth()` method!

```typescript
import request from 'supertest';
describe('Authentication & Authorization', () => {
    const API_URL = 'http://localhost:8080';
    it('1. Basic Authentication', async () => {
        // SuperTest has built-in support for Basic Auth
        const response = await request(API_URL)
            .get('/api/auth/basic')
            .auth('admin', 'admin'); // Pass username and password
        expect(response.status).toBe(200);
        expect(response.body.message).toBe('Basic Auth successful');
    });
```
*Note: Under the hood, `.auth('admin', 'admin')` automatically calculates the Base64 encoding and sets the `Authorization: Basic YWRtaW46YWRtaW4=` header for you!*

## 2. JWT Bearer Tokens (OAuth 2.0)

Modern APIs use stateless JSON Web Tokens (JWT). The typical flow is:
1. You send a `POST` request with your email and password to a `/login` endpoint.
2. The server responds with a secure JWT string.
3. You must attach this string as a `Bearer` token in the `Authorization` header of all subsequent requests.

Let's automate this exact flow:

```typescript
    let jwtToken: string; // Store state across tests
    it('2. Generate JWT Token (Login)', async () => {
        const payload = {
            email: 'admin@mycodeyatra.com',
            password: 'password123'
        };
        const response = await request(API_URL)
            .post('/api/auth/login')
            .send(payload);
        expect(response.status).toBe(200);
        expect(response.body).toHaveProperty('token');
        // Save the Bearer Token for the next test
        jwtToken = response.body.token;
    });
    it('3. Access Protected Route with Bearer Token', async () => {
        // Now we use the token we dynamically generated in the previous test!
        const response = await request(API_URL)
            .get('/api/auth/profile')
            .set('Authorization', `Bearer ${jwtToken}`); // Manually set header
        expect(response.status).toBe(200);
        expect(response.body.message).toBe('Welcome to your protected profile!');
    });
```

## 3. Testing Negative Authentication Scenarios

In security testing, the "Sad Path" is just as important as the "Happy Path". You must explicitly test that your API rejects unauthorized users!

```typescript
    it('4. Fail accessing Protected Route without Token', async () => {
        const response = await request(API_URL).get('/api/auth/profile');
        // Assert that the server strictly enforces 401 Unauthorized
        expect(response.status).toBe(401);
        expect(response.body.error).toBe('Missing token');
    });
});
```

## 4. Execution Results

Let's execute `npx jest tests/auth.test.ts` to watch our E2E Authentication flow run successfully against the server:

```bash
PASS tests/auth.test.ts
  Authentication & Authorization
    [PASS] 1. Basic Authentication (87 ms)
    [PASS] 2. Generate JWT Token (Login) (24 ms)
    [PASS] 3. Access Protected Route with Bearer Token (15 ms)
    [PASS] 4. Fail accessing Protected Route without Token (8 ms)
Test Suites: 1 passed, 1 total
Tests:       4 passed, 4 total
Snapshots:   0 total
Time:        4.466 s
```

## 5. Conclusion

By dynamically capturing Auth tokens and injecting them into headers via `.set()`, SuperTest can easily navigate enterprise-grade security layers.

Next up, we will look at how to test complex **GraphQL APIs** using SuperTest!
