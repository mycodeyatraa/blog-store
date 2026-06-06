---
title: Handling Request Timeouts & Chaos Testing
date: 28-Feb-2025
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [supertest, chaos-testing, timeout, error-handling, jest]
category: API Testing
categories: [API Testing, NodeJS, QA, Automation]
excerpt: >-
  Ensure your CI/CD pipeline never hangs again. Master SuperTest timeout controls and learn how to test against unexpected API failures.
readTime: 5 min read
---

No API is perfect. Networks fail, databases lock up, and servers randomly crash. If your automation suite only tests the "Happy Path" where everything responds in 50ms, your tests will fail miserably in a real-world CI/CD pipeline.

**Chaos Testing** is the practice of intentionally breaking things to ensure your application (and your test suite) handles failure gracefully. In this tutorial, we will use SuperTest to automate Chaos scenarios!

---

## 1. Simulating and Handling Timeouts

Imagine an endpoint that usually takes 50ms to respond suddenly takes 10 seconds. If your test suite doesn't have a timeout configured, it will hang indefinitely, blocking your entire deployment pipeline.

We can instruct SuperTest to forcibly abort a connection if the server takes too long using the `.timeout()` method.

```typescript
import request from 'supertest';
describe('Handling Request Timeouts & Chaos Testing', () => {
    const API_URL = 'http://localhost:8080';
    it('1. Simulate a Timeout Exception', async () => {
        // Our mock server will maliciously delay this response by 3000ms.
        // We instruct SuperTest to abort the connection if it takes longer than 1000ms.
        try {
            await request(API_URL)
                .get('/api/chaos/delay?ms=3000')
                .timeout({
                    response: 1000, // Wait max 1000ms for the server to start sending data
                    deadline: 1500  // Wait max 1500ms for the entire connection to complete
                });
            // If the code reaches here, the timeout failed!
            throw new Error('Test failed: The timeout did not trigger');
        } catch (error: any) {
            // Assert that SuperTest correctly aborted the connection and threw an error
            expect(error.message).toMatch(/timeout/i);
            expect(error.code).toBe('ECONNABORTED');
        }
    });
```

By explicitly wrapping the SuperTest call in a `try/catch` block, we can assert that the timeout exception was correctly thrown!

## 2. Handling Unexpected Content-Types

During a catastrophic server failure (like an Nginx 502 Bad Gateway), the server often responds with an HTML or XML error page instead of your expected JSON.

If your test simply asserts `expect(response.body.data).toBeDefined()`, it will throw a cryptic Type Error because `response.body` will be empty. 

Here is how you test that your API handles unexpected content boundaries:

```typescript
    it('2. Asserting against unexpected Content-Types (XML)', async () => {
        // Our mock server will intentionally return XML instead of JSON
        const response = await request(API_URL).get('/api/chaos/xml');
        expect(response.status).toBe(200);
        // 1. Assert that the server sent the wrong Content-Type
        expect(response.headers['content-type']).toContain('application/xml');
        // 2. SuperTest won't parse XML automatically into response.body!
        // We must parse `response.text` manually to read raw string payloads.
        expect(response.text).toContain('<name>Alice</name>');
    });
});
```

## 3. Execution Results

Let's execute `npx jest tests/chaos.test.ts`. Notice that the timeout test finishes in roughly ~1000ms, proving that SuperTest successfully aborted the connection before the server's 3000ms delay finished!

```bash
PASS tests/chaos.test.ts
  Handling Request Timeouts & Chaos Testing
    [PASS] 1. Simulate a Timeout Exception (1034 ms)
    [PASS] 2. Asserting against unexpected Content-Types (XML) (9 ms)
Test Suites: 1 passed, 1 total
Tests:       2 passed, 2 total
Snapshots:   0 total
Time:        4.481 s
```

## 4. Conclusion

Resilient automation suites don't just verify that APIs work; they verify that APIs fail *correctly*. 

Now that we have covered everything from WebSockets to Chaos Testing, we have effectively mastered the execution phase of SuperTest. But what about organizing this massive suite?

In our next blog, we will learn how to integrate **Faker.js** to dynamically generate thousands of random test permutations!
