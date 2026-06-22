---
title: "Using Hooks and Fixtures for Setup/Teardown"
date: "2025-03-06"
description: "Stop writing duplicate setup code. Learn how to use Playwright's hooks (beforeEach/afterEach) and built-in fixtures to keep your tests DRY."
tags: ["Playwright", "TypeScript", "Hooks", "Fixtures", "DRY"]
---

Welcome to Blog 22 of the **Playwright TypeScript Mastery Series**!

One of the foundational principles of software development is **DRY** (Don't Repeat Yourself). 

If you are writing an e-commerce test suite, almost every single test requires a user to be logged in. If you have 50 tests, do you copy and paste the login code 50 times? No! You use **Hooks**.

### Playwright Hooks

Playwright provides four primary hooks to manage your test lifecycles:
*   `test.beforeAll()`: Runs *once* before the entire test file starts.
*   `test.beforeEach()`: Runs before *every single test* in the file.
*   `test.afterEach()`: Runs after *every single test* (even if the test fails!).
*   `test.afterAll()`: Runs *once* after all tests in the file complete.

### Built-in Fixtures

In Playwright, you constantly see `({ page }) => { ... }`. `page` is a **Fixture**. 

Playwright provides built-in fixtures like `page` (an isolated browser tab) and `context` (an isolated browser session). The beauty of `beforeEach` is that the `page` fixture is passed into the hook, prepared, and then handed to your test!

Let's look at an example against our live practice site (`https://practice.mycodeyatra.com/#/login`):

Create `tests/blog22_hooks.spec.ts`:

```typescript
import { test, expect } from '@playwright/test';

test.describe('Blog 22: Hooks and Fixtures', () => {

  // The 'page' parameter here is a built-in Playwright "Fixture"
  test.beforeEach(async ({ page }) => {
    // This code runs BEFORE every single test in this block
    console.log('Running Setup: Logging in...');
    await page.goto('https://practice.mycodeyatra.com/#/login');
    await page.getByTestId('username').fill('admin');
    await page.getByTestId('password').fill('admin123');
    await page.getByTestId('login-btn').click();
    
    // Ensure we reached the profile page
    await expect(page).toHaveURL(/.*profile/);
  });

  test.afterEach(async ({ page }) => {
    // This code runs AFTER every single test in this block
    console.log('Running Teardown: Clearing session...');
    await page.evaluate(() => localStorage.clear());
  });

  test('Verify Dashboard Navigation', async ({ page }) => {
    // We don't need to write login code here! 
    const navText = page.locator('nav');
    await expect(navText).toBeVisible();
    console.log('Test 1: Dashboard navigation verified!');
  });

  test('Verify Profile Settings Accessible', async ({ page }) => {
    // Same here, we are already logged in!
    await page.goto('https://practice.mycodeyatra.com/#/profile');
    const header = page.locator('h2');
    await expect(header).toHaveText('Welcome back, admin!');
    console.log('Test 2: Profile settings accessible!');
  });

});
```

### Execution Output

When you run `npx playwright test tests/blog22_hooks.spec.ts`:

```bash
Running 2 tests using 1 worker

Running Setup: Logging in...
Test 1: Dashboard navigation verified!
Running Teardown: Clearing session...
  ✓  1 tests\blog22_hooks.spec.ts:27:7 › Blog 22: Hooks and Fixtures › Verify Dashboard Navigation (728ms)

Running Setup: Logging in...
Test 2: Profile settings accessible!
Running Teardown: Clearing session...
  ✓  2 tests\blog22_hooks.spec.ts:35:7 › Blog 22: Hooks and Fixtures › Verify Profile Settings Accessible (610ms)

  2 passed (2.6s)
```

Notice how `Running Setup` and `Running Teardown` wrap *both* tests independently!

### Conclusion

Using `beforeEach` is the standard way to handle repetitive setup actions like authenticating a user or navigating to a specific starting state. 

However, what if you have multiple files that all need this exact same login code? Copying the `beforeEach` block into 20 different files isn't DRY either.

In **Blog 23**, we will solve this by writing our own **Custom Fixtures**!
