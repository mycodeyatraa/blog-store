---
title: "Creating Custom Fixtures"
date: "2025-03-07"
description: "Take your test architecture to the next level. Learn how to extend Playwright to create your own custom, reusable fixtures like pre-authenticated pages."
tags: ["Playwright", "TypeScript", "Custom Fixtures", "Architecture"]
---

Welcome to Blog 23 of the **Playwright TypeScript Mastery Series**!

In our last blog, we used `test.beforeEach()` to log into our application. This works great for a single file, but what happens when you have 50 different test files that all require the user to be logged in? 

Copying `test.beforeEach()` into every file is an architectural nightmare. 

The Playwright solution is **Custom Fixtures**.

### What is a Custom Fixture?

When you type `async ({ page }) => {}`, you are requesting the `page` fixture from Playwright. What if we could tell Playwright to give us an entirely new fixture called `loggedInPage` that is *already logged in* before the test even starts?

We can achieve this by extending the base `test` object.

### Building `loggedInPage`

Let's test this against our live login page (`https://practice.mycodeyatra.com/#/login`).

Create `tests/blog23_custom_fixtures.spec.ts`:

```typescript
import { test as base, expect } from '@playwright/test';

// 1. Define the type for our custom fixtures
type MyFixtures = {
  loggedInPage: import('@playwright/test').Page;
};

// 2. Extend the base test to include our custom fixture
export const test = base.extend<MyFixtures>({
  loggedInPage: async ({ page }, use) => {
    // Setup: Everything before `use()` runs before the test
    console.log('[Fixture Setup] Navigating and logging in...');
    await page.goto('https://practice.mycodeyatra.com/#/login');
    await page.getByTestId('username').fill('admin');
    await page.getByTestId('password').fill('admin123');
    await page.getByTestId('login-btn').click();
    await expect(page).toHaveURL(/.*profile/); // Wait for successful login
    
    // 3. Yield the prepared page to the test
    await use(page);
    
    // Teardown: Everything after `use()` runs after the test finishes
    console.log('[Fixture Teardown] Clearing local storage...');
    await page.evaluate(() => localStorage.clear());
  },
});

// 4. Use our custom fixture in a test!
test.describe('Blog 23: Custom Fixtures', () => {
  
  // Notice we use { loggedInPage } instead of { page }
  test('Verify Profile Page with Custom Fixture', async ({ loggedInPage }) => {
    // We are ALREADY logged in when this test starts!
    const header = loggedInPage.locator('h2');
    await expect(header).toHaveText('Welcome back, admin!');
    console.log('Test 1 Passed: Used custom authenticated fixture!');
  });

  // If a test doesn't need to be logged in, it can just use the standard { page }
  test('Verify Login Page without Custom Fixture', async ({ page }) => {
    await page.goto('https://practice.mycodeyatra.com/#/login');
    const header = page.locator('h2');
    await expect(header).toHaveText('Sign In');
    console.log('Test 2 Passed: Used standard page fixture!');
  });

});
```

### Execution Output

When you run `npx playwright test tests/blog23_custom_fixtures.spec.ts`:

```text
Running 2 tests using 1 worker

[Fixture Setup] Navigating and logging in...
Test 1 Passed: Used custom authenticated fixture!
[Fixture Teardown] Clearing local storage...
  OK   1 tests/blog23_custom_fixtures.spec.ts:32:3 > Blog 23: Custom Fixtures > Verify Profile Page with Custom Fixture (722ms)

Test 2 Passed: Used standard page fixture!
  OK   2 tests/blog23_custom_fixtures.spec.ts:40:3 > Blog 23: Custom Fixtures > Verify Login Page without Custom Fixture (505ms)

  2 passed (2.5s)
```

### The Power of `use()`

The magic happens at `await use(page)`. 
1. Everything before `use()` acts as your `beforeEach`.
2. `use()` pauses the fixture and executes your actual test block.
3. Once the test finishes, execution resumes and everything after `use()` acts as your `afterEach`.

By centralizing this code into a fixture, you can now import this custom `test` object into *any* file in your framework and instantly request a `loggedInPage`.

In **Blog 24**, we will explore another architectural pattern for code reuse: the **Page Object Model (POM)**!
