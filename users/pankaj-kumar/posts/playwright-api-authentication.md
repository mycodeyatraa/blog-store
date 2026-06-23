---
title: "Authentication in API Testing"
date: "2025-03-22"
description: "Learn how to handle Bearer Token authorization and cookie-based session management in Playwright API tests, keeping credentials secure."
tags: ["Playwright", "TypeScript", "API Testing", "Authentication"]
---

Welcome to Blog 39 of the **Playwright TypeScript Mastery Series**!

Most production REST APIs are secured behind some form of authentication. To successfully test protected endpoints, your automation suite must supply authentication credentials.

Today, we will learn how to handle authentication in Playwright API tests, covering both **token-based authentication** (Bearer Tokens) and **cookie-based session authentication**.

---

### Common API Authentication Methods

1. **Token-Based (Bearer Tokens)**: The client sends a header like `Authorization: Bearer <token>` with every request.
2. **Cookie-Based**: The client sends a unique identifier in the `Cookie` header (e.g., `sessionId=xyz123`) which the server correlates to an active session.

---

### Step 1: Implementing the Test Spec

Create a test file `tests/blog39_api_auth.spec.ts` to implement our authentication handling flows:

```typescript
import { test, expect, playwright } from '@playwright/test';

test.describe('Blog 39: Authentication in API Testing', () => {

  test('Token-Based Authentication (Bearer Token)', async ({ request }) => {
    // 1. Fetch your API token (usually from process.env for security)
    const apiToken = process.env.API_TOKEN || 'secret-token-12345';

    // 2. Send request with the Authorization header
    const response = await request.get('https://jsonplaceholder.typicode.com/posts/1', {
      headers: {
        'Authorization': `Bearer ${apiToken}`
      }
    });

    expect(response.status()).toBe(200);
    console.log('Bearer token auth request executed successfully.');
  });

  test('Cookie-Based Authentication', async ({ playwright }) => {
    // 1. Create a custom APIRequestContext with injected cookies
    const authenticatedContext = await playwright.request.newContext({
      baseURL: 'https://jsonplaceholder.typicode.com',
      extraHTTPHeaders: {
        // Send session information via cookies
        'Cookie': 'sessionId=abc123xyz789; loggedIn=true'
      }
    });

    // 2. Send request within this authenticated context
    const response = await authenticatedContext.get('/posts/1');
    expect(response.status()).toBe(200);

    console.log('Cookie-based auth request executed successfully.');

    // 3. Dispose of the custom context
    await authenticatedContext.dispose();
  });

});
```

---

### Best Practices for Securing API Credentials

To ensure security in your test automation framework:
- **Never Hardcode Secrets**: Avoid checking tokens, keys, or passwords directly into version control.
- **Use Environment Variables**: Load secrets using environment variables (e.g., via `process.env.API_TOKEN`).
- **Load variables securely**: Combine this with a logging framework that masks authorization headers so sensitive headers do not appear in console outputs or test reports.

---

### Step 2: Run the Test

Execute the test via Playwright CLI:

```
npx playwright test tests/blog39_api_auth.spec.ts
```

**Output:**

```
Running 2 tests using 1 worker

Bearer token auth request executed successfully.
  ✓  1 tests/blog39_api_auth.spec.ts:5:7 › Blog 39: Authentication in API Testing › Token-Based Authentication (Bearer Token) (127ms)
Cookie-based auth request executed successfully.
  ✓  2 tests/blog39_api_auth.spec.ts:20:7 › Blog 39: Authentication in API Testing › Cookie-Based Authentication (34ms)

  2 passed (1.3s)
```

In the next blog, we will cover **Hybrid Testing (Web + API)** and learn how to speed up UI tests by using API endpoints to set login states!
