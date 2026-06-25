---
title: Advanced Allure: Epic Mapping and Historical Trends
date: 07-May-2025
lastUpdated: 07-May-2025
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: ["playwright", "typescript", "allure", "advanced", "agile", "ci-cd"]
category: Reporting
categories: ["Reporting", "UI Automation", "Playwright", "TypeScript", "Allure"]
excerpt: >-
  Elevate your Playwright reporting by mapping tests to Agile Epics, capturing dynamic data-driven parameters, and rendering historical trend graphs in CI/CD.
readTime: 5 min read
---

In the previous tutorial, we added basic Allure tags and screenshots. However, to truly utilize Allure as an Enterprise Dashboard, we must configure it to track historical trends, map tests to Agile business requirements, and dynamically record environment variables.

### 1. Behavior-Driven Hierarchy (Epic/Feature/Story)

If your Product Owner asks, "What is our coverage for the Checkout feature?", handing them a list of 500 flat test names is useless. 

Allure allows you to map your Playwright tests directly into Agile hierarchies. When you do this, the dashboard unlocks the **"Behaviors"** tab, visually grouping your execution results into a tree structure!

**File:** `tests/allure-advanced.spec.ts`

```typescript
import { test } from '@playwright/test';
import { allure } from 'allure-playwright';
 
test.describe('E-Commerce Checkout', () => {
 
  // Apply hierarchy to all tests in this block
  test.beforeEach(async () => {
    await allure.epic('Web Application');
    await allure.feature('Checkout Flow');
    await allure.story('Credit Card Processing');
  });
 
  test('Guest user can checkout using valid credit card', async ({ page }) => {
    // ... test logic ...
  });
});
```

### 2. Dynamic Parameters

If you run Data-Driven Tests (DDT) or cross-browser execution, a failing test named "Login Test" isn't helpful if you don't know *what* data caused the failure.

You can explicitly inject parameters into the Allure report:

```typescript
test('Guest user can checkout using valid credit card', async ({ page }) => {
  // Inject the exact variables that drove this execution
  await allure.parameter('Browser', 'Chromium');
  await allure.parameter('Card Type', 'Visa');
  await allure.parameter('Environment', process.env.ENV || 'Staging');
  
  // ... test logic ...
});
```

### 3. Tracking Historical Trends (CI/CD)

The most powerful feature of Allure is the **Trend Graph**. It shows you if your test suite is getting flakier or more stable over time.

By default, every time you run Playwright locally, Allure generates a fresh report. To build historical trends, Allure needs to see the *previous* run's history data before generating the new report. 

When running in CI/CD (like GitHub Actions or Jenkins), you must implement this workflow:

1. Execute Playwright tests (outputs to `allure-results/`).
2. Download the `allure-report/history/` folder from the *previous* CI pipeline run.
3. Copy that `history/` folder *into* your current `allure-results/` folder.
4. Run `allure generate`.

```bash
# Example CI/CD Script Logic:
npx playwright test
 
# (Assume you downloaded previous history into ./old-history)
cp -r ./old-history ./allure-results/history
 
# Generate the final report with the trend graph unlocked!
npx allure generate allure-results --clean
```

### 4. Setting Environment Properties

You can display high-level environment data (OS, Node version, Base URL) on the main Allure dashboard widget by writing an `environment.properties` file into the results folder *after* the tests finish but *before* generating the report.

```bash
echo "BaseURL=https://staging.app.com" > allure-results/environment.properties
echo "Browser=Playwright WebKit" >> allure-results/environment.properties
npx allure generate allure-results
```

### Summary

By deeply integrating `allure-playwright`, your automation repository acts as a single source of truth. Engineers get exact line-level failure context, and Product Owners get beautiful, high-level business charts grouped by Epic and Feature!
