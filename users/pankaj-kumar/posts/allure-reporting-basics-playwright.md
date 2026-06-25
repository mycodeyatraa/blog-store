---
title: Allure Reporting Basics in Playwright
date: 06-May-2025
lastUpdated: 06-May-2025
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: ["playwright", "typescript", "allure", "reporting", "dashboard", "testing"]
category: Reporting
categories: ["Reporting", "UI Automation", "Playwright", "TypeScript", "Allure"]
excerpt: >-
  Transform your Playwright test execution data into visually stunning, enterprise-grade interactive dashboards using the Allure Framework.
readTime: 5 min read
---

When your automated test suite expands from 50 to 500 tests, relying on the terminal output or the default HTML reporter is no longer sufficient. Enterprise teams need dashboards that track historical trends, categorizations, failure categories, and severity metrics.

Enter **Allure Framework**. Allure is an industry-standard open-source reporting tool designed to create visually stunning, highly detailed test execution reports.

Let's integrate it into our Playwright TypeScript framework!

### 1. Installation

You need the Playwright reporter plugin and the Allure command-line tool (to generate and serve the dashboard).

```bash
npm i -D allure-playwright allure-commandline
```

### 2. Configuration

Add the Allure reporter to your `playwright.config.ts`. You can run multiple reporters simultaneously!

**File:** `playwright.config.ts`

```typescript
import { defineConfig } from '@playwright/test';
 
export default defineConfig({
  reporter: [
    ['html'], // Keep the default HTML reporter
    ['allure-playwright', { 
      detail: true,
      outputFolder: 'allure-results',
      suiteTitle: false 
    }]
  ],
});
```

### 3. Annotating Tests with Metadata

Allure shines when you enrich your tests with metadata. You can tag tests with Severities, JIRA Links, and precise Categories directly inside your `.spec.ts` files using the `allure` object.

**File:** `tests/allure-basics.spec.ts`

```typescript
import { test, expect } from '@playwright/test';
import { allure } from 'allure-playwright';
 
test('Basic login validation with Allure metadata', async ({ page }) => {
  // 1. Add Allure Metadata for the Report Dashboard
  await allure.owner('QA Automation Team');
  await allure.tags('Smoke', 'Authentication', 'Critical');
  await allure.severity('blocker');
  await allure.issue('JIRA-1234', 'https://jira.company.com/browse/JIRA-1234');
  
  // 2. Define structured Test Steps to make the report human-readable
  await test.step('Navigate to the Login Page', async () => {
    await page.goto('/login');
  });
 
  await test.step('Enter User Credentials', async () => {
    await page.fill('#username', 'test_user');
    await page.fill('#password', 'secure_pass!');
  });
 
  await test.step('Submit Form and Assert Dashboard Access', async () => {
    await page.click('#submit');
    await expect(page.locator('.welcome')).toBeVisible();
  });
 
  // 3. Attachments: Add screenshots or API payloads directly to the report
  await test.step('Attach Evidence', async () => {
    const mockApiPayload = JSON.stringify({ status: 'success' }, null, 2);
    // This payload will appear as a downloadable file inside the Allure UI!
    await allure.attachment('API Response Payload', mockApiPayload, 'application/json');
  });
});
```

### 4. Generating and Viewing the Report

When you run your tests, Playwright will now output raw XML/JSON data into an `allure-results` folder.

```bash
# 1. Run the tests to generate raw data
npx playwright test
```

To view the actual dashboard, you use the Allure CLI to compile the raw results into a beautiful HTML site and serve it locally:

```bash
# 2. Compile and open the dashboard
npx allure serve allure-results
```

### Summary

By adding `allure-playwright` and wrapping logic inside `test.step()`, you instantly transform a wall of terminal text into an interactive, enterprise-grade dashboard complete with JIRA links, severities, and downloadable test artifacts!
