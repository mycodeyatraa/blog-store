---
title: "Validating GET APIs"
date: "2025-03-19"
description: "Learn how to send HTTP GET requests with query parameters, validate response headers, parse response bodies, and perform schema/property assertions in Playwright."
tags: ["Playwright", "TypeScript", "API Testing", "Validation"]
---

Welcome to Blog 36 of the **Playwright TypeScript Mastery Series**!

HTTP GET requests are the most common operations when interacting with RESTful services. They retrieve resource representations from the server.

Today, we will learn how to automate GET requests in Playwright. We'll cover sending query parameters, validating response headers, parsing JSON response arrays, and writing robust assertions against the body data.

---

### Key Concepts of GET Request Validation

When testing a GET endpoint, you should validate the following:
1. **HTTP Status Code**: Ensuring the server returned `200 OK` (or other expected statuses like `404 Not Found`).
2. **Response Headers**: Verifying `content-type` is correct (typically `application/json`).
3. **Data Types**: Ensuring arrays are actually arrays and fields match the correct formats.
4. **Data Properties**: Iterating through collection arrays or checking individual object values.

---

### Step 1: Implementing the Test Spec

Create a test file `tests/blog36_api_get.spec.ts` to implement our GET API validation flow:

```typescript
import { test, expect } from '@playwright/test';

test.describe('Blog 36: Validating GET APIs in Playwright', () => {

  test('GET Request with Query Parameters', async ({ request }) => {
    // 1. Send GET request with query parameters (e.g., /posts?userId=1)
    const response = await request.get('https://jsonplaceholder.typicode.com/posts', {
      params: {
        userId: 1
      }
    });

    // 2. Validate the response status is 200 OK
    expect(response.status()).toBe(200);
    expect(response.ok()).toBeTruthy(); // Returns true for statuses in the 200-299 range

    // 3. Validate response headers
    const headers = response.headers();
    expect(headers['content-type']).toContain('application/json');

    // 4. Validate the response JSON body
    const posts = await response.json();
    
    // Assert that the response is an array
    expect(Array.isArray(posts)).toBeTruthy();
    
    // Assert on array length
    expect(posts.length).toBeGreaterThan(0);
    
    // Verify each post in the array belongs to userId 1 and has essential keys
    posts.forEach(post => {
      expect(post.userId).toBe(1);
      expect(post).toHaveProperty('id');
      expect(post).toHaveProperty('title');
      expect(post).toHaveProperty('body');
    });

    console.log(`Successfully verified ${posts.length} posts for userId 1.`);
  });

});
```

---

### Passing Query Parameters Natively

Instead of manually constructing query strings like `?userId=1&active=true`, Playwright provides a native `params` object parameter in the request configuration block:

```typescript
const response = await request.get('/posts', {
  params: {
    userId: 1,
    active: true
  }
});
```

Playwright automatically encodes and appends these parameters to the request URL.

---

### Step 2: Run the Test

Execute the test via Playwright CLI:

```
npx playwright test tests/blog36_api_get.spec.ts
```

**Output:**

```
Running 1 test using 1 worker

Successfully verified 10 posts for userId 1.
  ✓  1 tests/blog36_api_get.spec.ts:5:7 › Blog 36: Validating GET APIs in Playwright › GET Request with Query Parameters (806ms)

  1 passed (1.9s)
```

In the next blog, we will cover **POST APIs in Playwright** and learn how to configure payloads, handle request types, and validate created resources!
