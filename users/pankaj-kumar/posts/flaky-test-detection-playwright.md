---
title: Conquering Flaky Tests in Playwright
date: 11-May-2025
lastUpdated: 11-May-2025
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: ["playwright", "typescript", "reporting", "flaky-tests", "retries", "ci-cd"]
category: Reporting
categories: ["Reporting", "UI Automation", "Playwright", "TypeScript", "Best Practices"]
excerpt: >-
  Master CI/CD stability by configuring smart retries in Playwright, and learn how to dynamically tag flaky executions for dashboard visibility using `test.info().retry`.
readTime: 4 min read
---

A "flaky test" is the worst enemy of an automation team. It is a test that fails due to a timeout or a race condition, but when you run it again, it miraculously passes.

If pipelines constantly fail due to flakes, developers will stop looking at the results entirely. We must manage flakiness systematically.

### 1. Enabling Retries in Playwright

The easiest way to stabilize a pipeline is to configure Playwright to automatically retry failing tests. A test is only marked as a "failure" if it exhausts all of its retry attempts.

Usually, you only want retries enabled in your CI/CD pipeline, not during local development.

**File:** `playwright.config.ts`

```typescript
import { defineConfig } from '@playwright/test';
 
export default defineConfig({
  // Retry failing tests twice on CI, zero times locally
  retries: process.env.CI ? 2 : 0,
});
```

You can also override retries for a specific, notoriously unstable test block:

```typescript
test.describe.configure({ retries: 5 }); // 5 retries for this block only!
```

### 2. Detecting and Tagging Flaky Tests

If a test fails on Attempt 1 but passes on Attempt 2, Playwright considers the overall run "Passed". While this keeps the pipeline green, it creates a blind spot: you still have a flaky test hiding in your codebase!

To fix this, we can dynamically check the `test.info().retry` property. If it's greater than zero, it means the test flaked, and we should tag it!

**File:** `tests/flaky.spec.ts`

```typescript
import { test, expect } from '@playwright/test';
 
test('Test prone to network flakiness', async ({ page }) => {
  // ... perform unstable test actions ...
  
  // After the test succeeds, check if we had to retry to get here!
  if (test.info().retry > 0) {
    
    // Tag it for the built-in HTML Reporter
    test.info().annotations.push({
      type: 'flake-detected',
      description: `Test passed but required ${test.info().retry} retries.`
    });
 
    // Or if you use Allure, you can explicitly tag it:
    // await allure.tags('FLAKY');
  }
  
  expect(true).toBeTruthy();
});
```

### 3. Reviewing Flakes in Reports

Because we tagged the flaky test, we can now easily find it.
- **In the HTML Reporter:** A specific "Flaky" column will appear on the dashboard.
- **In Allure:** If you use `allure-playwright`, Allure has a native "Flaky" badge (a little bomb icon) that explicitly marks tests that have a history of passing and failing sporadically.

### Summary

Retries are a necessary evil in UI automation to combat network latency and animation delays. However, you must always **log your retries**. By dynamically tagging tests where `retry > 0`, you ensure your CI pipeline stays green for developers, while providing a clear backlog of unstable tests for QA to fix!
