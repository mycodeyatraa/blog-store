---
title: "APIRequestContext Deep Dive"
date: "2025-03-18"
description: "Learn how to use, configure, and manually instantiate Playwright's APIRequestContext to handle multiple base URLs, custom headers, and dynamic session states in API tests."
tags: ["Playwright", "TypeScript", "API Testing", "Framework"]
---

Welcome to Blog 35 of the **Playwright TypeScript Mastery Series**!

In modern end-to-end automation, testing APIs is just as crucial as testing user interfaces. Playwright handles API testing natively through `APIRequestContext`.

Today, we will dive deep into `APIRequestContext`. We'll explore how the built-in `request` fixture works, how to configure global API settings, and how to programmatically spin up custom, isolated API contexts at runtime.

---

### What is APIRequestContext?

An `APIRequestContext` is a container that handles HTTP/HTTPS requests. It manages its own cookies, headers, and request settings. 

In Playwright, you can interact with `APIRequestContext` in two ways:
1. **The Default `request` Fixture**: A built-in context that inherits base settings from your `playwright.config.ts`.
2. **Programmatic Creation**: Spinning up a brand new, isolated request context manually at runtime using the `playwright` object.

---

### Step 1: Configuring the Global Context in `playwright.config.ts`

To set up base options for the default `request` fixture, you configure the `use` block in your `playwright.config.ts`:

```typescript
import { defineConfig } from '@playwright/test';
 
export default defineConfig({
  use: {
    // Defines base URL for all API requests using the default fixture
    baseURL: 'https://jsonplaceholder.typicode.com',
    extraHTTPHeaders: {
      'Accept': 'application/json',
      'Content-Type': 'application/json',
    },
  },
});
```

---

### Step 2: Implementing the Spec Code

Let's write a spec file `tests/blog35_api_request_context.spec.ts` that demonstrates both the default context and programmatic creation.

```typescript
import { test, expect } from '@playwright/test';
 
test.describe('Blog 35: APIRequestContext Deep Dive', () => {
 
  test('Demonstrating default request context configuration', async ({ request }) => {
    // The default 'request' fixture automatically inherits headers and baseURL
    const response = await request.get('https://jsonplaceholder.typicode.com/posts/1');
    expect(response.status()).toBe(200);
    
    const body = await response.json();
    console.log(`[Default Context] Title: ${body.title}`);
  });
 
  test('Creating and using a custom APIRequestContext programmatically', async ({ playwright }) => {
    // 1. Create an isolated custom request context with custom headers and base URL
    const customContext = await playwright.request.newContext({
      baseURL: 'https://jsonplaceholder.typicode.com',
      extraHTTPHeaders: {
        'Authorization': 'Bearer MY_SECRET_AUTH_TOKEN',
        'Accept': 'application/json',
        'Custom-Header': 'PlaywrightDemo'
      }
    });
 
    // 2. Send requests relative to the custom baseURL
    const response = await customContext.get('/posts/2');
    expect(response.status()).toBe(200);
    
    const body = await response.json();
    console.log(`[Custom Context] Title: ${body.title}`);
 
    // 3. Clean up and dispose of the custom context to release system resources
    await customContext.dispose();
  });
 
});
```

---

### When to Create Custom Contexts Programmatically?

Spinning up custom request contexts is useful when:
- Your tests need to talk to **multiple distinct domains** (e.g., a payment gateway API, a user auth service, and your main app service).
- You need to simulate **multiple concurrent users** (e.g., User A sending a request with token X, and User B sending a request with token Y).
- You need to dynamically generate or rotation-check auth headers mid-test without changing the global configuration.

---

### Critical Lifecycle Rule: Disposing Contexts

When you request the default `{ request }` fixture, Playwright automatically disposes of it when the test ends.

However, when you create a custom context programmatically via `playwright.request.newContext()`, **you are responsible for its lifecycle**. You must call `await customContext.dispose()` at the end of your test (or in an `afterEach`/`afterAll` hook). This ensures cookies, connections, and memory buffers are freed up.

---

### Step 3: Run the Test

Execute the spec using the Playwright CLI:

```
npx playwright test tests/blog35_api_request_context.spec.ts
```

**Output:**

```
Running 2 tests using 1 worker
 
[Default Context] Title: sunt aut facere repellat provident occaecati excepturi optio reprehenderit
  ✓  1 tests/blog35_api_request_context.spec.ts:5:7 › Blog 35: APIRequestContext Deep Dive › Demonstrating default request context configuration (112ms)
[Custom Context] Title: qui est esse
  ✓  2 tests/blog35_api_request_context.spec.ts:13:7 › Blog 35: APIRequestContext Deep Dive › Creating and using a custom APIRequestContext programmatically (679ms)
 
  2 passed (1.9s)
```

In the next blog, we will cover **GET APIs in Playwright** and learn how to manage query parameters, headers, and response parsing!
