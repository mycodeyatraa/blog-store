---
title: "API Testing Capabilities in Playwright"
date: "2025-03-09"
description: "Playwright isn't just for UI testing. Learn how to use Playwright's built-in APIRequestContext to send HTTP requests, validate API responses, and setup test data directly."
tags: ["Playwright", "TypeScript", "API Testing", "HTTP Request"]
---

Welcome to Blog 25 of the **Playwright TypeScript Mastery Series**!

Many automation engineers assume Playwright is strictly a browser automation tool like Selenium. However, Playwright is actually a full-stack testing framework!

It comes with a built-in `APIRequestContext` that allows you to send HTTP requests completely independent of the browser. This is incredible for two reasons:
1. **API Testing**: You can replace Postman or RestAssured with pure Playwright code.
2. **Data Setup**: Instead of clicking through the UI to create test data, you can send a lightning-fast API request to seed the database *before* your UI test starts.

### The `request` Fixture

To interact with APIs, we ask Playwright for the `request` fixture instead of the `page` fixture.

Let's test this against a public mock API (`jsonplaceholder.typicode.com`).

Create `tests/blog25_api_testing.spec.ts`:

```typescript
import { test, expect } from '@playwright/test';

test.describe('Blog 25: API Testing Capabilities', () => {

  // Notice we request the { request } fixture instead of { page }
  test('Sending a GET request and validating the response', async ({ request }) => {
    // 1. Send an HTTP GET request to a public API
    const response = await request.get('https://jsonplaceholder.typicode.com/posts/1');
    
    // 2. Validate the HTTP Status Code
    expect(response.status()).toBe(200);
    expect(response.ok()).toBeTruthy(); // .ok() means status is in the 200-299 range
    
    // 3. Extract the JSON body from the response
    const responseBody = await response.json();
    
    // 4. Assert against the JSON data
    expect(responseBody.id).toBe(1);
    expect(responseBody).toHaveProperty('title');
    expect(responseBody).toHaveProperty('body');
    
    console.log('API Test Passed! Fetched Post Title:', responseBody.title);
  });

  test('Creating data using a POST request', async ({ request }) => {
    // 1. Send an HTTP POST request with a JSON payload
    const response = await request.post('https://jsonplaceholder.typicode.com/posts', {
      data: {
        title: 'Playwright API Test',
        body: 'This is incredibly fast.',
        userId: 1,
      }
    });

    // 2. Validate it was created successfully (HTTP 201 Created)
    expect(response.status()).toBe(201);
    
    const responseBody = await response.json();
    expect(responseBody.title).toBe('Playwright API Test');
    
    console.log('Successfully created a post via API!');
  });
});
```

### Execution Output

When you run `npx playwright test tests/blog25_api_testing.spec.ts`:

```text
Running 2 tests using 1 worker

API Test Passed! Fetched Post Title: sunt aut facere repellat provident occaecati excepturi optio reprehenderit
  ✓  1 tests\blog25_api_testing.spec.ts:6:7 › Blog 25: API Testing Capabilities › Sending a GET request and validating the response (120ms)
Successfully created a post via API!
  ✓  2 tests\blog25_api_testing.spec.ts:25:7 › Blog 25: API Testing Capabilities › Creating data using a POST request (687ms)

  2 passed (2.0s)
```

Notice how fast it is? Because there is no browser involved, API tests run in milliseconds!

### Conclusion

Playwright's API capabilities allow you to blur the line between UI and API testing. You can write a single script that uses `request.post()` to instantly create a user in the backend, and then uses `page.goto()` to log in as that user in the UI.

In **Blog 26**, we will dive into a crucial topic for stable tests: **Handling Network Interception and Mocking Responses**!
