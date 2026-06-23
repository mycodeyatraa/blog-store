---
title: "Validating POST APIs"
date: "2025-03-20"
description: "Learn how to automate HTTP POST requests in Playwright, send JSON payloads, configure headers, and validate resource creation responses."
tags: ["Playwright", "TypeScript", "API Testing", "Validation"]
---

Welcome to Blog 37 of the **Playwright TypeScript Mastery Series**!

HTTP POST requests are used to submit data to the server, typically creating new resources. Unlike GET requests, POST requests carry a message payload (body) containing the resource configuration.

Today, we will learn how to automate POST requests in Playwright, configure JSON payloads, set headers, assert on successful resource creation (`201 Created`), and verify response body values.

---

### Key Concepts of POST Request Validation

When testing a POST endpoint, you should validate the following:
1. **HTTP Status Code**: Ensuring the server returned `201 Created` (representing a successfully created resource).
2. **Dynamic ID Generation**: Verifying that the response includes a newly generated identifier (e.g. `id`).
3. **Payload Parity**: Asserting that the properties returned in the response match the payload fields we sent.

---

### Step 1: Implementing the Test Spec

Create a test file `tests/blog37_api_post.spec.ts` to implement our POST API validation flow:

```typescript
import { test, expect } from '@playwright/test';

test.describe('Blog 37: Validating POST APIs in Playwright', () => {

  test('POST Request with JSON Payload', async ({ request }) => {
    // 1. Define the request payload (body)
    const requestPayload = {
      title: 'Automating APIs with Playwright',
      body: 'Playwright makes API testing simple and extremely fast.',
      userId: 1
    };

    // 2. Send the POST request with the payload in the 'data' parameter
    const response = await request.post('https://jsonplaceholder.typicode.com/posts', {
      data: requestPayload,
      headers: {
        'Content-Type': 'application/json; charset=UTF-8'
      }
    });

    // 3. Validate status code is 201 Created
    expect(response.status()).toBe(201);
    expect(response.ok()).toBeTruthy();

    // 4. Validate the response body content
    const responseBody = await response.json();
    
    // Ensure the response contains the auto-generated resource ID
    expect(responseBody).toHaveProperty('id');
    
    // Validate that the returned data matches the payload we sent
    expect(responseBody.title).toBe(requestPayload.title);
    expect(responseBody.body).toBe(requestPayload.body);
    expect(responseBody.userId).toBe(requestPayload.userId);

    console.log(`Successfully created resource. Returned ID: ${responseBody.id}`);
  });

});
```

---

### Sending Payloads Natively

Playwright simplifies sending request payloads through the `data` option in the request configuration block:

```typescript
const response = await request.post('/posts', {
  data: {
    title: 'Example',
    body: 'Content'
  }
});
```

When you pass an object to the `data` parameter, Playwright automatically:
- Converts the JavaScript object into a JSON string using `JSON.stringify()`.
- Sets the `Content-Length` header.
- Sets the `Content-Type` header to `application/json` (unless overridden in the `headers` block).

---

### Step 2: Run the Test

Execute the test via Playwright CLI:

```
npx playwright test tests/blog37_api_post.spec.ts
```

**Output:**

```
Running 1 test using 1 worker

Successfully created resource. Returned ID: 101
  ✓  1 tests/blog37_api_post.spec.ts:5:7 › Blog 37: Validating POST APIs in Playwright › POST Request with JSON Payload (730ms)

  1 passed (1.9s)
```

In the next blog, we will cover **PUT, PATCH, and DELETE APIs** and learn how to update and remove resources in Playwright!
