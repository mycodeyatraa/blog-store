---
title: "Assertions with Playwright Test"
date: "2025-02-19"
description: "Learn how to validate application state using the built-in expect library. Discover the difference between Hard and Soft assertions, and understand auto-retrying matchers."
tags: ["Playwright", "TypeScript", "Assertions", "Expect", "Testing"]
---

Welcome to Blog 7 of the **Playwright TypeScript Mastery Series**!

An automation script that just clicks buttons without validating anything is not a test. A test requires **Assertions**.

In the Java and Selenium world, you usually have to import a third-party assertion library like TestNG or JUnit to validate your data. In Playwright, everything is built directly into the `@playwright/test` runner via the globally available `expect` function.

Today, we will learn about Auto-retrying matchers, Generic matchers, and the incredibly useful Soft Assertions.

### 1. Auto-Retrying Matchers (Web Assertions)

The biggest cause of flaky tests is asserting an element's state *before* the browser has finished rendering it.

Playwright's web-specific matchers (like `toBeVisible()`, `toHaveText()`, `toBeEnabled()`) are designed to be **auto-retrying**. This means if you assert an element is visible, but it's currently hidden, Playwright won't instantly crash. It will automatically poll the DOM and wait (up to your configured timeout, usually 5 seconds) for the condition to become true!

Create `tests/blog7_assertions.spec.ts` in your project:

```typescript
import { test, expect } from '@playwright/test';
test('Auto-retrying Hard Assertions', async ({ page }) => {
  await page.goto('https://practice.mycodeyatra.com/#/login');
  const formHeader = page.getByText('Login Portal');
  // Playwright will wait up to 5 seconds for this to become visible
  await expect(formHeader).toBeVisible();
  const usernameInput = page.locator('input#username');
  await expect(usernameInput).toBeEmpty();
  await usernameInput.fill('admin');
  await expect(usernameInput).toHaveValue('admin');
});
```
*Note: You **must** use `await` with auto-retrying matchers, because Playwright is actively polling the browser!*

### 2. Generic Matchers

Sometimes, you are not testing the UI. You might be testing an API response code, or doing some mathematical validation in TypeScript. 

For these, you use **Generic Matchers**. These do *not* poll the DOM, and they do *not* require the `await` keyword.

```typescript
test('Non-retrying Generic Assertions', async ({ page }) => {
  const statusCode = 200;
  // Generic matchers do NOT use 'await'
  expect(statusCode).toBe(200);
  expect([1, 2, 3]).toContain(2);
  expect('Playwright TypeScript').toContain('Type');
});
```

### 3. Hard vs Soft Assertions

By default, all `expect()` assertions are **Hard Assertions**. If a Hard Assertion fails on Line 10, the test immediately crashes, and Line 11 is never executed. 

But what if you are validating a massive table with 15 columns? If Column 2 is wrong, you still want to check Columns 3 through 15 so you can file a complete bug report!

This is where **Soft Assertions** come in. If a Soft Assertion fails, Playwright logs the error but *continues executing the test*. The test will only formally be marked as "Failed" at the very end.

```typescript
test('Soft Assertions', async ({ page }) => {
  await page.goto('https://practice.mycodeyatra.com/#/login');
  const loginButton = page.getByRole('button', { name: 'Login' });
  // Use expect.soft() to prevent immediate crashes
  await expect.soft(loginButton).toBeEnabled();
  await expect.soft(loginButton).toHaveText('Login');
  // Even if the text was wrong above, this code will still run!
  console.log('Test execution continued perfectly!');
});
```

### Execution Output

When you run `npx playwright test tests/blog7_assertions.spec.ts`:

```
Running 3 tests using 1 worker
  ✓  1 Auto-retrying Hard Assertions (1.1s)
  ✓  2 Non-retrying Generic Assertions (0.1s)
Test execution continued perfectly!
  ✓  3 Soft Assertions (0.9s)
  3 passed (2.1s)
```

### Conclusion

You now have the power to validate application states with rock-solid reliability. By leaning on Playwright's auto-retrying matchers, you eliminate the need for brittle `sleep()` commands, and by using `expect.soft()`, you can gather comprehensive failure data in a single run.

In **Blog 8**, we will put all the pieces together and write our **First E2E Test with Playwright Test**!
