---
title: Automated Failure Analysis: Categorizing Pipeline Errors
date: 10-May-2025
lastUpdated: 10-May-2025
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: ["playwright", "typescript", "reporting", "failure-analysis", "debugging", "ci-cd"]
category: Reporting
categories: ["Reporting", "UI Automation", "Playwright", "TypeScript", "Best Practices"]
excerpt: >-
  Stop guessing why pipelines are red. Learn how to systematically manage known bugs using `test.fail()` and intercept raw stack traces to auto-categorize failures.
readTime: 4 min read
---

When a CI/CD pipeline turns red because 20 tests failed, the immediate question from Engineering Leadership is: *"Are these real bugs, or is the automation just flaky?"*

**Failure Analysis** is the practice of systematically categorizing *why* tests fail, so teams can prioritize their investigation. A test that fails due to a genuine UI assertion error requires immediate developer attention. A test that fails due to a 502 Bad Gateway error requires DevOps attention.

### 1. Handling Known Bugs

The worst thing an automation team can do is ignore a failing pipeline because "we know about that bug." If the pipeline is always red, people stop trusting it.

If a test is failing due to a known, ticketed issue, use `test.fail()`. This tells Playwright to *expect* a failure. If the test fails, the pipeline stays green! Even better, if a developer fixes the bug quietly and the test suddenly passes, Playwright will throw an error telling you to remove the `test.fail()` annotation!

**File:** `tests/failure-analysis.spec.ts`

```typescript
import { test, expect } from '@playwright/test';
 
test('Checkout fails when using expired promo code', async ({ page }) => {
  // Mark this as a known failure. The pipeline will NOT go red if this test fails.
  test.fail(true, 'Known Bug: JIRA-999 - Backend incorrectly accepts expired promo codes');
  
  await page.goto('/checkout');
  await page.fill('#promo', 'EXPIRED2020');
  await page.click('#apply');
  
  // The backend is bugged, so this error DOES NOT appear, causing the test to fail.
  await expect(page.locator('.error')).toHaveText('Promo code expired.');
});
```

### 2. Auto-Categorizing Failures at Runtime

We can use the `test.afterEach()` hook to inspect the raw Error Stack Trace of a failing test before the runner shuts down. By performing string matching on the error message, we can automatically categorize the failure and attach that metadata to our HTML or Allure reports!

```typescript
test.afterEach(async ({ }, testInfo) => {
  if (testInfo.status === 'failed') {
    const errorStr = testInfo.error?.message || '';
 
    // Default category
    let failureCategory = 'UNKNOWN';
 
    // 1. Analyze the stack trace for specific keywords
    if (errorStr.includes('Timeout')) {
      failureCategory = 'TIMEOUT_FLAKE';
    } 
    else if (errorStr.includes('Expected')) {
      failureCategory = 'ASSERTION_ERROR'; // Likely a genuine product bug!
    }
    else if (errorStr.includes('net::ERR_CONNECTION_REFUSED')) {
      failureCategory = 'ENVIRONMENT_DOWN';
    }
 
    // 2. Attach the category to the Playwright/Allure Report
    testInfo.annotations.push({ 
      type: 'failure-category', 
      description: failureCategory 
    });
 
    // 3. (Optional) Dispatch an immediate Slack alert if the environment is down!
    if (failureCategory === 'ENVIRONMENT_DOWN') {
      console.error('CRITICAL: Test environment is offline. Alerting DevOps...');
      // await fetch('https://hooks.slack.com/services/T00000...', { ... });
    }
  }
});
```

### Summary

By explicitly managing known bugs with `test.fail()` and automatically intercepting Stack Traces to categorize errors, you eliminate the "boy who cried wolf" syndrome. Your pipelines remain trustworthy, and engineers instantly know whether they are looking at a flaky test, an environment outage, or a critical production bug!
