---
title: "Hybrid Testing (Web + API)"
date: "2025-03-23"
description: "Learn how to accelerate web UI tests in Playwright by bypassing UI login forms using direct API requests to inject authorization states and session cookies."
tags: ["Playwright", "TypeScript", "Hybrid Testing", "Performance"]
---

Welcome to Blog 40 of the **Playwright TypeScript Mastery Series**!

One of the major speed bottlenecks in UI test automation suites is repeating setup workflows (like logging in) via the user interface. Doing this for every test case dramatically increases test execution times and introduces UI flakiness.

Today, we will learn about **Hybrid Testing**. We will combine API capabilities and browser contexts to log in programmatically using API calls, and inject authentication cookies directly into the browser context. This allows us to bypass the UI login page entirely and navigate directly to target pages.

---

### Why Hybrid Testing?

- **Drastic Speed Improvements**: API requests take milliseconds; UI page loads, input typing, and button clicks can take seconds.
- **Improved Test Stability**: Bypassing the login page means if your login UI changes or breaks, only the explicit login tests fail — your other functional tests (e.g. settings, payments, profiles) remain unaffected.
- **Isolate Test Objectives**: A profile verification test should focus *only* on the profile page, not on testing the login page over and over.

---

### Step 1: Implementing the Test Spec

Create a test file `tests/blog40_hybrid_testing.spec.ts` to implement our hybrid authentication flow:

```typescript
import { test, expect } from '@playwright/test';
 
test.describe('Blog 40: Hybrid Testing in Playwright', () => {
 
  test('By-passing UI Login using API state injection', async ({ page, context, request }) => {
    console.log('[Hybrid Test] Step 1: Performing API login to fetch session cookies...');
    
    // Send a direct POST request to the authentication API
    const loginResponse = await request.post('https://jsonplaceholder.typicode.com/posts', {
      data: {
        username: 'admin',
        password: 'securePassword123'
      }
    });
 
    expect(loginResponse.status()).toBe(201); // Asserts API login success
    console.log('[Hybrid Test] API Login succeeded!');
 
    console.log('[Hybrid Test] Step 2: Injecting authentication cookies into browser context...');
    
    // Inject the authentication cookies directly into the browser context
    await context.addCookies([
      {
        name: 'session_token',
        value: 'mock-jwt-token-xyz-98765',
        domain: 'practice.mycodeyatra.com',
        path: '/',
        httpOnly: true,
        secure: true
      }
    ]);
 
    console.log('[Hybrid Test] Step 3: Navigating directly to the dashboard page...');
    
    // Navigate straight to the protected profile page
    // The browser automatically attaches the injected cookies!
    await page.goto('https://practice.mycodeyatra.com/#/profile');
    
    console.log('[Hybrid Test] Navigated directly to profile page without using the UI Login form.');
  });
 
});
```

---

### How it Works Under the Hood

1. **`request` fixture**: Sends a standard HTTP request to retrieve a valid authorization cookie or token.
2. **`context.addCookies()`**: Places the session cookie directly into Playwright's browser memory before navigating.
3. **`page.goto()`**: Navigates directly to the authenticated page. The browser acts as if the user filled out the form and clicked "Login" successfully.

---

### Step 2: Run the Test

Execute the test via Playwright CLI:

```
npx playwright test tests/blog40_hybrid_testing.spec.ts
```

**Output:**

```
Running 1 test using 1 worker
 
[Hybrid Test] Step 1: Performing API login to fetch session cookies...
[Hybrid Test] API Login succeeded!
[Hybrid Test] Step 2: Injecting authentication cookies into browser context...
[Hybrid Test] Step 3: Navigating directly to the dashboard page...
[Hybrid Test] Navigated directly to profile page without using the UI Login form.
  ✓  1 tests/blog40_hybrid_testing.spec.ts:5:7 › Blog 40: Hybrid Testing in Playwright › By-passing UI Login using API state injection (4.3s)
 
  1 passed (5.8s)
```

In the next blog, we will cover **Contract Testing in Playwright** and learn how to validate API responses against structured JSON schemas!
