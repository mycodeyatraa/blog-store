---
title: "Custom Playwright Test Reporters"
date: "2025-03-17"
description: "Learn how to build a custom test reporter in Playwright to format execution metrics, track test status, and output styled logs to the console."
tags: ["Playwright", "TypeScript", "Reporting", "Framework"]
---

Welcome to Blog 34 of the **Playwright TypeScript Mastery Series**!

While Playwright comes with excellent built-in reporters (like HTML, Line, Dot, and List), enterprise test frameworks often require custom reporting. Whether you need to push results to a database, send real-time alerts to Slack, or print customized, branded logs to the terminal, writing a **Custom Reporter** is the cleanest solution.

Today, we will build a custom console reporter that outputs formatted execution metrics and detailed errors.

---

### Understanding the Reporter Interface

Playwright exposes a `Reporter` interface under `@playwright/test/reporter`. It contains lifecycle hooks that fire at specific points of the execution run:

- **`onBegin`**: Fired once before running the test suite.
- **`onTestBegin`**: Fired when a single test starts running.
- **`onTestEnd`**: Fired when a single test completes (passing, failing, or skipped).
- **`onEnd`**: Fired once after all tests have completed.

---

### Step 1: Create the Custom Reporter

Create a new file called `utils/custom-reporter.ts` and implement the `Reporter` interface:

```typescript
import { Reporter, TestCase, TestResult, FullResult, FullConfig, Suite } from '@playwright/test/reporter';
 
class CustomReporter implements Reporter {
  onBegin(config: FullConfig, suite: Suite) {
    console.log(`\n==================================================`);
    console.log(`[CUSTOM REPORTER] Starting Test Suite Run`);
    console.log(`[CUSTOM REPORTER] Total tests in this run: ${suite.allTests().length}`);
    console.log(`==================================================\n`);
  }
 
  onTestBegin(test: TestCase, result: TestResult) {
    console.log(`[TEST START] Running: "${test.title}" ...`);
  }
 
  onTestEnd(test: TestCase, result: TestResult) {
    const duration = result.duration;
    const status = result.status.toUpperCase();
    console.log(`[TEST END] Finished: "${test.title}" | Status: ${status} | Duration: ${duration}ms`);
    
    if (result.errors && result.errors.length > 0) {
      console.log(`[TEST ERROR] Details for "${test.title}":`);
      result.errors.forEach((err, idx) => {
        console.log(`  ${idx + 1}. ${err.message}`);
      });
    }
    console.log('--------------------------------------------------');
  }
 
  onEnd(result: FullResult) {
    console.log(`\n==================================================`);
    console.log(`[CUSTOM REPORTER] Test Suite Run Completed!`);
    console.log(`[CUSTOM REPORTER] Overall Status: ${result.status.toUpperCase()}`);
    console.log(`==================================================\n`);
  }
}
 
export default CustomReporter;
```

---

### Step 2: Write a Demo Test Spec

Let's create a spec file `tests/blog34_reporter.spec.ts` with one passing and one failing test case to see our reporter log different statuses:

```typescript
import { test, expect } from '@playwright/test';
 
test.describe('Blog 34: Custom Playwright Test Reporters', () => {
 
  test('Passing test execution', async ({ page }) => {
    console.log('[TEST LOG] Navigating to login page...');
    await page.goto('https://practice.mycodeyatra.com/#/login');
    const header = page.getByText('Login Portal');
    await expect(header).toBeVisible();
    console.log('[TEST LOG] Login header is visible.');
  });
 
  test('Failing test execution for reporter logging demonstration', async ({ page }) => {
    console.log('[TEST LOG] Attempting to navigate and check non-existent element...');
    await page.goto('https://practice.mycodeyatra.com/#/login');
    
    const nonExistentElement = page.locator('#non-existent-button-id');
    await expect(nonExistentElement).toBeVisible({ timeout: 2000 });
  });
 
});
```

---

### Step 3: Run Tests using the Custom Reporter

You can trigger the custom reporter by passing the `--reporter` CLI flag and pointing to your reporter script:

```
npx playwright test tests/blog34_reporter.spec.ts --reporter=./utils/custom-reporter.ts
```

**Output:**

```
==================================================
[CUSTOM REPORTER] Starting Test Suite Run
[CUSTOM REPORTER] Total tests in this run: 2
==================================================
 
[TEST START] Running: "Passing test execution" ...
[TEST END] Finished: "Passing test execution" | Status: PASSED | Duration: 4210ms
--------------------------------------------------
[TEST START] Running: "Failing test execution for reporter logging demonstration" ...
[TEST END] Finished: "Failing test execution for reporter logging demonstration" | Status: FAILED | Duration: 2546ms
[TEST ERROR] Details for "Failing test execution for reporter logging demonstration":
  1. Error: expect(locator).toBeVisible() failed
 
Locator: locator('#non-existent-button-id')
Expected: visible
Timeout: 2000ms
Error: element(s) not found
 
Call log:
  - Expect "toBeVisible" with timeout 2000ms
  - waiting for locator('#non-existent-button-id')
--------------------------------------------------
 
==================================================
[CUSTOM REPORTER] Test Suite Run Completed!
[CUSTOM REPORTER] Overall Status: FAILED
==================================================
```

---

### Configuring the Custom Reporter Permanently

Instead of passing the `--reporter` flag every time, you can register your reporter permanently in your `playwright.config.ts` file:

```typescript
import { defineConfig } from '@playwright/test';
 
export default defineConfig({
  reporter: [
    ['list'], // You can run standard reporters alongside your custom one!
    ['./utils/custom-reporter.ts']
  ],
});
```

In the next blog, we will step into **Phase 5** and start exploring **API Testing** in Playwright using `APIRequestContext`!
