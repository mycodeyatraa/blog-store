---
title: "Configuration Management in Playwright"
date: "2025-03-13"
description: "Learn how to master the playwright.config.ts file to manage global setups, timeouts, multiple environments, and reporter configurations for enterprise scaling."
tags: ["Playwright", "TypeScript", "Configuration", "Framework"]
---

Welcome to Blog 29 of the **Playwright TypeScript Mastery Series**!

Up until now, we have been writing tests and running them with the default Playwright settings. However, in an enterprise environment, you rarely want to run tests against a single hardcoded URL or with a single browser. 

You need to run tests against **QA**, **Staging**, and **Production** environments. You need to configure global timeouts, setup API authentication before tests start, and generate complex HTML reports.

All of this is managed through the central nervous system of Playwright: the `playwright.config.ts` file.

### Exploring `playwright.config.ts`

When you initialize a Playwright project, a `playwright.config.ts` file is generated in the root directory. Let's look at a customized enterprise-ready configuration:

```typescript
import { defineConfig, devices } from '@playwright/test';

// Read environment variables (e.g., QA, STAGE, PROD)
const env = process.env.TEST_ENV || 'QA';

// Define base URLs dynamically based on the environment
const baseUrlMap = {
  QA: 'https://practice.mycodeyatra.com',
  STAGE: 'https://stage.mycodeyatra.com',
  PROD: 'https://mycodeyatra.com'
};

export default defineConfig({
  // Directory where all tests are located
  testDir: './tests',
  
  // Run tests in files in parallel
  fullyParallel: true,
  
  // Fail the build on CI if you accidentally left test.only in the source code
  forbidOnly: !!process.env.CI,
  
  // Retry failing tests twice on CI, but zero times locally
  retries: process.env.CI ? 2 : 0,
  
  // Opt out of parallel tests on CI.
  workers: process.env.CI ? 1 : undefined,
  
  // Generate HTML and JSON reports
  reporter: [['html'], ['json', { outputFile: 'results.json' }]],
  
  // Global Timeout for the entire test run
  globalTimeout: 60 * 60 * 1000, // 1 hour

  // Shared settings for all the projects below
  use: {
    // Dynamic Base URL injected into all tests
    baseURL: baseUrlMap[env],

    // Collect trace when retrying the failed test
    trace: 'on-first-retry',
    
    // Take a screenshot on test failure
    screenshot: 'only-on-failure',
    
    // Default explicit wait timeout for locators
    actionTimeout: 10000,
  },

  // Configure projects for major browsers
  projects: [
    {
      name: 'chromium',
      use: { ...devices['Desktop Chrome'] },
    },
    {
      name: 'firefox',
      use: { ...devices['Desktop Firefox'] },
    },
  ],
});
```

### Leveraging the Configuration in Tests

Once your `playwright.config.ts` is configured, you no longer need to hardcode URLs in your tests!

Create `tests/blog29_config.spec.ts`:

```typescript
import { test, expect } from '@playwright/test';

test.describe('Blog 29: Configuration Management', () => {

  test('Using Environment Variables for URLs', async ({ page }) => {
    // We can use process.env to read dynamically injected variables
    const baseUrl = process.env.BASE_URL || 'https://practice.mycodeyatra.com';
    
    console.log(`Navigating to environment URL: ${baseUrl}`);
    
    await page.goto(baseUrl);
    
    // Assert we reached the correct page
    await expect(page.locator('h1').first()).toBeVisible();
    console.log('Successfully navigated using configuration!');
  });

});
```

### Execution Output

When you run `npx playwright test tests/blog29_config.spec.ts`:

```
Running 1 test using 1 worker

Navigating to environment URL: https://practice.mycodeyatra.com
Successfully navigated using configuration!
  OK  1 tests/blog29_config.spec.ts:8:7 > Blog 29: Configuration Management > Using Environment Variables for URLs (4.6s)

  1 passed (6.1s)
```

### Passing Variables from the Terminal

If you want to run the tests against the STAGE environment, you can inject the environment variable directly from your command line:

**Mac/Linux:**

```
TEST_ENV=STAGE npx playwright test
```

**Windows (PowerShell):**

```
$env:TEST_ENV="STAGE"; npx playwright test
```

### Conclusion

Mastering the `playwright.config.ts` file is essential for scaling your automation framework. It allows you to separate test logic from environmental data, making your codebase extremely modular and robust.

In **Blog 30**, we will begin designing our **Logging Framework** to capture custom step-by-step logs during execution!
