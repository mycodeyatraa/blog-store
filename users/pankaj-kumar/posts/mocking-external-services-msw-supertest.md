---
title: Mocking External Services with MSW (Mock Service Worker) in Supertest
date: 02-Mar-2026
lastUpdated: 02-Mar-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: ["supertest", "msw", "mocking", "api-testing", "express"]
category: API Supertest
categories: ["API Supertest", "Node.js", "Express"]
excerpt: >-
  Isolate microservices under test! Integrate Mock Service Worker (MSW) to stub third-party APIs during Supertest integration tests.
readTime: 8 min read
---

# Mocking External Services with MSW (Mock Service Worker) in Supertest

When testing Express API endpoints that communicate with external 3rd-party services (e.g., Stripe Payment Gateway, SendGrid Email API, Twilio SMS), hitting live external APIs in integration test runs introduces slowness, cost, and rate-limit flakiness.

**Mock Service Worker (MSW)** intercepts outgoing Node.js `http` and `fetch` requests at the network level, returning deterministic mock responses without modifying application source code.

---

## 1. Setting Up MSW in Node.js

```bash
npm install --save-dev msw
```

---

## 2. Defining MSW Request Handlers

**mocks/handlers.ts**

```typescript
import { http, HttpResponse } from 'msw';
 
export const handlers = [
  // Intercept 3rd-party Stripe Payment API
  http.post('https://api.stripe.com/v1/charges', () => {
    return HttpResponse.json({
      id: 'ch_mock_12345',
      status: 'succeeded',
      amount: 4999,
      currency: 'usd'
    }, { status: 200 });
  })
];
```

---

## 3. Configuring MSW Server Lifecycle with Jest & Supertest

**tests/payment-integration.spec.ts**

```typescript
import request from 'supertest';
import { setupServer } from 'msw/node';
import { handlers } from '../mocks/handlers';
import app from '../src/app';
 
const server = setupServer(...handlers);
 
beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());
 
describe('Express Payment Endpoint with MSW Network Interception', () => {
 
  test('should complete checkout order via mocked Stripe API', async () => {
    const response = await request(app)
      .post('/api/v1/checkout')
      .send({ cartId: 'cart_999', paymentMethodToken: 'tok_visa' });
 
    expect(response.status).toBe(200);
    expect(response.body.orderStatus).toBe('PAID');
    expect(response.body.chargeId).toBe('ch_mock_12345');
  });
});
```
