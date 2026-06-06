---
title: Handling Async/Await & Promises
date: 23-Feb-2025
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [supertest, async, await, promises, jest]
category: API Testing
categories: [API Testing, NodeJS, Automation, TypeScript]
excerpt: >-
  Prevent flaky tests by mastering NodeJS asynchronous execution. Learn how to handle Promises, Async/Await, and parallel HTTP requests in SuperTest.
readTime: 5 min read
---

Because NodeJS is fundamentally single-threaded and non-blocking, every single HTTP Request fired by SuperTest operates asynchronously. 

If you do not correctly handle the asynchronous nature of NodeJS, your Jest tests will complete before the HTTP responses even return, leading to false positives and highly flaky test suites!

In this tutorial, we will explore how to manage SuperTest execution flow using both legacy Promises and modern `async/await`.

---

## 1. The Legacy Way: Returning Promises

In older versions of JavaScript, we handled asynchronous operations using `.then()` and `.catch()` chains.

If you choose to write tests this way in Jest, **you must return the Promise** to the test runner. If you forget the `return` keyword, Jest will assume the test passed instantly without waiting for the `.then()` assertions to finish.

```typescript
import request from 'supertest';
describe('Handling Async/Await & Promises', () => {
    const API_URL = 'http://localhost:8080';
    it('1. Traditional Promises (The Old Way)', () => {
        // We MUST return the Promise in Jest if we don't use async/await
        return request(API_URL)
            .get('/api/health')
            .then(response => {
                expect(response.status).toBe(200);
            })
            .catch(error => {
                throw error;
            });
    });
});
```

## 2. The Modern Way: Async / Await

Introduced in ES8, `async/await` is syntactic sugar over Promises that allows us to write asynchronous code that *looks* synchronous. It completely eliminates "callback hell" and deeply nested `.then()` blocks.

```typescript
    it('2. Modern Async/Await (The New Way)', async () => {
        // Notice the 'async' keyword in the function signature!
        try {
            // Execution pauses here until the API responds
            const response = await request(API_URL).get('/api/health');
            // Assertions run after response is received
            expect(response.status).toBe(200);
        } catch (error) {
            console.error('API Request failed', error);
            throw error;
        }
    });
```
This is the universally recommended approach for writing SuperTest automation today.

## 3. High Performance: Parallel Requests

What if you want to test 5 different independent endpoints and verify their performance? Using `await` sequentially would take a long time. 

Instead, we can fire all requests simultaneously and wait for them to finish in parallel using `Promise.all`:

```typescript
    it('3. Parallel Requests with Promise.all', async () => {
        // Fire multiple API requests simultaneously
        const request1 = request(API_URL).get('/api/health');
        const request2 = request(API_URL).get('/api/users');
        // Wait for both to complete in parallel
        const [healthResponse, usersResponse] = await Promise.all([request1, request2]);
        expect(healthResponse.status).toBe(200);
        expect(usersResponse.status).toBe(200);
    });
```

## 4. Execution Results

Let's run `npx jest tests/async.test.ts` to see our tests execute. Notice how fast the parallel request test finishes compared to the traditional ones!

```bash
PASS tests/async.test.ts
  Handling Async/Await & Promises
    [PASS] 1. Traditional Promises (The Old Way) (52 ms)
    [PASS] 2. Modern Async/Await (The New Way) (9 ms)
    [PASS] 3. Parallel Requests with Promise.all (8 ms)
Test Suites: 1 passed, 1 total
Tests:       3 passed, 3 total
Snapshots:   0 total
Time:        3.811 s
```

## 5. Next Steps

Mastering the Event Loop and `async/await` is crucial for writing fast, reliable NodeJS automation. Now that we understand how to correctly pause execution and handle payloads, it's time to dive into security.

In our next article, we will tackle **Authentication & Authorization** testing with SuperTest!
