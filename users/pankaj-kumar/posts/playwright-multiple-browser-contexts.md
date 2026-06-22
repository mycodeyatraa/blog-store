---
title: "Testing Multiple Browser Contexts"
date: "2025-03-11"
description: "Need to test a chat application or multi-user workflow? Learn how to spawn and manage multiple isolated browser contexts simultaneously."
tags: ["Playwright", "TypeScript", "Browser Context", "Multi-user"]
---

Welcome to Blog 27 of the **Playwright TypeScript Mastery Series**!

One of Playwright's most powerful architectural features is its ability to easily handle **multiple browser contexts**. 

A browser context is like an "Incognito" or "Private" window. It has its own cookies, local storage, and session data. If you log into a website in Context A, you will remain logged out in Context B.

This is absolutely essential for testing multi-user applications like:
*   Chat applications (User A sends a message, User B receives it)
*   Approval workflows (Employee submits a request, Manager approves it)
*   Gaming or collaborative editing tools

### Spawning Multiple Contexts

Let's write a script that opens two separate contexts simultaneously against our practice site.

Create `tests/blog27_browser_contexts.spec.ts`:

```typescript
import { test, expect } from '@playwright/test';

test.describe('Blog 27: Multiple Browser Contexts', () => {

  // Notice we request the { browser } fixture instead of { page }
  test('Simulating a multi-user workflow', async ({ browser }) => {
    
    // 1. Create two completely isolated browser sessions
    const adminContext = await browser.newContext();
    const userContext = await browser.newContext();

    // 2. Open pages in each context
    const adminPage = await adminContext.newPage();
    const userPage = await userContext.newPage();

    // 3. Admin logs in
    await adminPage.goto('https://practice.mycodeyatra.com/#/login');
    await adminPage.getByTestId('username').fill('admin');
    await adminPage.getByTestId('password').fill('admin123');
    await adminPage.getByTestId('login-btn').click();
    await expect(adminPage).toHaveURL(/.*profile/);

    // 4. Regular User visits the home page (unauthenticated)
    await userPage.goto('https://practice.mycodeyatra.com');
    const header = userPage.locator('h1').first();
    await expect(header).toHaveText(/Technical Insights Built for/);
    
    // Prove isolation: Admin is on the profile page, User is on the home page!
    console.log('Admin Page URL:', adminPage.url());
    console.log('User Page URL:', userPage.url());
    
    // 5. Clean up manually since we created these contexts ourselves
    await adminContext.close();
    await userContext.close();
  });

});
```

### Execution Output

When you run `npx playwright test tests/blog27_browser_contexts.spec.ts`:

```text
Running 1 test using 1 worker

Admin Page URL: https://practice.mycodeyatra.com/#/profile
User Page URL: https://practice.mycodeyatra.com/
  OK   1 tests/blog27_browser_contexts.spec.ts:6:7 > Blog 27: Multiple Browser Contexts > Simulating a multi-user workflow (4.3s)

  1 passed (5.5s)
```

Playwright handled both sessions completely independently within the same script!

### Why Not Use `{ page }`?

If you request the default `{ page }` fixture from Playwright, it automatically spins up a context, creates a page, and gives it to you. That's why you don't usually need to call `browser.newContext()`. 

However, when you need strict control over multiple parallel users, dropping down to the `{ browser }` fixture gives you the power to spawn as many isolated contexts as your RAM can handle.

In **Blog 28**, we will cover the topic of **Visual Regression Testing**!
