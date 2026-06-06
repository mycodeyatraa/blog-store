---
title: Introduction to SuperTest & Express.js
date: 19-Feb-2025
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [supertest, nodejs, jest, typescript, api-testing]
category: API Testing
categories: [API Testing, NodeJS, Automation, TypeScript]
excerpt: >-
  Kickstart your NodeJS API Testing journey! Learn what SuperTest is, why it's the industry standard, and write your first asynchronous GET request using Jest and TypeScript.
readTime: 4 min read
---

Welcome to the brand new masterclass series on **NodeJS API Testing with SuperTest**!

While Java developers rely on RestAssured, the NodeJS ecosystem has universally adopted **SuperTest** as the gold standard for HTTP assertion. Whether you are building an Express API, a NestJS microservice, or serverless functions, SuperTest provides a fluent API for testing your endpoints.

---

## 1. What is SuperTest?

SuperTest is a high-level abstraction built on top of `SuperAgent`. It allows you to simulate HTTP requests against your NodeJS API and effortlessly chain assertions to verify the response.

It seamlessly integrates with testing frameworks like **Jest**, **Mocha**, and **Vitest**.

## 2. Why Choose SuperTest?

1. **Framework Agnostic:** It works out of the box with Express, Koa, Fastify, and NextJS.
2. **Internal API Testing:** It can test your Express app directly in-memory without even starting an HTTP port!
3. **Fluent Chainable API:** Write assertions that read like plain English.
4. **TypeScript Support:** Native typings for powerful code autocomplete.

## 3. Writing our First Test

In this series, we will be using **Jest** and **TypeScript**. Let's write a simple test that hits an external API Gateway and verifies its health status.

```typescript
import request from 'supertest';
describe('Introduction to SuperTest', () => {
    // The base URL of our API
    const API_URL = 'http://localhost:8080';
    it('should fetch the health status of the API', async () => {
        // 1. Arrange & Act: Send a GET Request
        const response = await request(API_URL).get('/api/health');
        // 2. Assert the HTTP Status Code
        expect(response.status).toBe(200);
        // 3. Assert the JSON Response Body
        expect(response.body).toHaveProperty('status', 'UP');
        expect(response.body).toHaveProperty('uptime');
    });
});
```

### Breaking down the code:
1. `request(API_URL)` binds SuperTest to our target server.
2. `.get('/api/health')` sends an HTTP GET request asynchronously.
3. `expect(response.status)` uses Jest's assertion library to verify the HTTP status.

## 4. Testing Negative Scenarios

It's equally important to test how your API handles failure. What happens when a user requests an endpoint that doesn't exist? Let's write a negative test:

```typescript
    it('should fail when accessing an invalid endpoint', async () => {
        const response = await request(API_URL).get('/api/invalid-endpoint');
        // Assert that we get a 404 Not Found
        expect(response.status).toBe(404);
    });
```

## 5. Next Steps

In this introductory blog, we covered the basics of sending GET requests to a live URL using SuperTest and Jest. 

In the next article, we will set up the actual Jest and TypeScript configurations in `package.json` so you can execute these tests in your local development environment!
