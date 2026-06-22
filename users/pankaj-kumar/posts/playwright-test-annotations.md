---
title: "Playwright Test Annotations (skip, only, fail)"
date: "2025-03-05"
description: "Master your test suite execution. Learn how to isolate specific tests, skip broken tests, and mark failing tests as expected using Playwright annotations."
tags: ["Playwright", "TypeScript", "Annotations", "Test Runner"]
---

Welcome to Blog 21 of the **Playwright TypeScript Mastery Series**!

We have covered how to write tests, but how do we *manage* them? 

What do you do when a test is broken but you don't want to delete the code? What if you have 500 tests in a file, but you only want to debug one specific test? What if there is a known bug in the application, and you want the test to fail *without* failing your entire CI/CD pipeline?

Playwright **Annotations** solve all of this.

### The Core Annotations

Let's test these annotations against our login page (`https://practice.mycodeyatra.com/#/login`). 

Create `tests/blog21_annotations.spec.ts`:

```typescript
import { test, expect } from '@playwright/test';

test.describe('Blog 21: Test Annotations', () => {

  // 1. skip()
  // Use this when a test is broken and you want to temporarily disable it
  // without deleting the code.
  test.skip('This test is currently broken', async ({ page }) => {
    await page.goto('https://practice.mycodeyatra.com/#/login');
    // This code will NOT execute
    await expect(page.locator('h2')).toHaveText('Broken Login'); 
  });

  // 2. fail()
  // Use this when you KNOW a test will fail (e.g., a known bug).
  // If the test FAILS, Playwright marks it as a "success" (expected failure).
  // If the test actually PASSES, Playwright will yell at you to remove the fail annotation!
  test('This test is expected to fail due to a known bug', async ({ page }) => {
    test.fail(); // Marks the test as an expected failure
    
    await page.goto('https://practice.mycodeyatra.com/#/login');
    // We expect the title to be "Sign In", but we assert it is "Wrong Title"
    await expect(page.locator('h2')).toHaveText('Wrong Title');
  });

  // 3. fixme()
  // Identical to skip(), but acts as a semantic flag for developers 
  // that this test requires maintenance.
  test.fixme('This test needs to be updated', async ({ page }) => {
    await page.goto('https://practice.mycodeyatra.com/#/login');
  });

  // 4. Standard Passing Test
  test('This test will run normally', async ({ page }) => {
    await page.goto('https://practice.mycodeyatra.com/#/login');
    await expect(page.locator('h2')).toHaveText('Sign In');
  });
  
  // NOTE: If we used test.only('Name', ...) on any test here, 
  // Playwright would ignore all other tests in the entire suite and only run that one!
});
```

### Execution Output

When you run `npx playwright test tests/blog21_annotations.spec.ts`, you will see something very interesting:

```text
Running 4 tests using 1 worker

  -  1 tests/blog21_annotations.spec.ts:8:8 > Blog 21: Test Annotations > This test is currently broken
  ✘  2 tests/blog21_annotations.spec.ts:18:7 > Blog 21: Test Annotations > This test is expected to fail due to a known bug (5.6s)
  -  3 tests/blog21_annotations.spec.ts:29:8 > Blog 21: Test Annotations > This test needs to be updated
  OK   4 tests/blog21_annotations.spec.ts:34:7 > Blog 21: Test Annotations > This test will run normally (529ms)

  2 skipped
  2 passed (7.3s)
```

Notice the summary: **`2 passed`**! 

Wait, the second test failed, right? Why did Playwright pass the suite? Because we used `test.fail()`. The test failed *as expected*, so Playwright considers the test run a success. 

### Conclusion

Annotations are your best friend for debugging (`.only()`) and maintaining a clean test report (`.skip()`, `.fail()`) when dealing with known application bugs or flaky code.

In **Blog 22**, we will learn one of the most powerful architectural features of Playwright: **Custom Fixtures**!
