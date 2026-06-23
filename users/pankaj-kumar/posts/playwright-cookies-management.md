---
title: "Cookies Management"
date: "2025-04-07"
description: "Learn how to add, inspect, and clear browser cookies at the context level in Playwright TypeScript."
tags: ["Playwright", "TypeScript", "Cookies", "addCookies", "clearCookies"]
---

Welcome to Blog 55 of the **Playwright TypeScript Mastery Series**!

Cookies play a vital role in user session state tracking, authentication persistence, marketing consent flow, and target personalization. During automation runs, you often need to mock cookies (like bypass cookies or auth sessions) or verify that security tags (`HttpOnly`, `Secure`) are set correctly on specific cookies.

Today, we will cover context-level cookie management using Playwright's `BrowserContext` APIs.

---

### Understanding the Cookie Structure in Playwright

Cookies in Playwright are managed at the `BrowserContext` level. When you define or query a cookie, it contains several key attributes:
- `name` (string): The cookie name.
- `value` (string): The cookie value.
- `domain` (string): The domain the cookie belongs to (e.g. `example.com`).
- `path` (string): The path the cookie applies to (typically `/`).
- `expires` (number): Unix epoch timestamp in seconds.
- `httpOnly` (boolean): Restricts JavaScript access to the cookie.
- `secure` (boolean): Restricts the cookie to HTTPS transmissions only.

---

### Step 1: Implementing the Cookies Spec

Create a test file `tests/blog55_cookies.spec.ts`:

```typescript
import { test, expect } from '@playwright/test';
 
test.describe('Blog 55: Cookies Management in Playwright', () => {
 
  test('Add, retrieve and clear browser cookies', async ({ context, page }) => {
    // 1. Define and add custom cookies to the browser context
    const testCookie = {
      name: 'auth_session',
      value: 'token_abc123',
      domain: 'example.com',
      path: '/',
      httpOnly: true,
      secure: true,
      expires: Math.floor(Date.now() / 1000) + 3600 // 1 hour expiration
    };
 
    await context.addCookies([testCookie]);
 
    // Navigate to the domain where the cookie is active
    await page.goto('https://example.com');
 
    // 2. Retrieve cookies for this domain
    const activeCookies = await context.cookies('https://example.com');
    const authCookie = activeCookies.find(c => c.name === 'auth_session');
 
    expect(authCookie).toBeDefined();
    expect(authCookie?.value).toBe('token_abc123');
    console.log(`[Cookies] Active Cookie Value: ${authCookie?.value}`);
 
    // 3. Clear cookies and verify
    await context.clearCookies();
    const remainingCookies = await context.cookies('https://example.com');
    const clearedAuthCookie = remainingCookies.find(c => c.name === 'auth_session');
 
    expect(clearedAuthCookie).toBeUndefined();
  });
 
});
```

---

### Step 2: Running the Test

Execute the test via Playwright CLI:

```
npx playwright test tests/blog55_cookies.spec.ts
```

**Output:**

```
Running 1 test using 1 worker
 
[Cookies] Active Cookie Value: token_abc123
  ✓  1 tests/blog55_cookies.spec.ts:5:7 › Blog 55: Cookies Management in Playwright › Add, retrieve and clear browser cookies (280ms)
 
  1 passed (1.7s)
```

In the next blog, we will learn how to handle multi-tab/window testing workflows by managing **multiple Pages and Tabs** inside a single Browser Context!
