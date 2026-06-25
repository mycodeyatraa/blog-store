---
title: Scaling Playwright with LambdaTest Cloud Grids
date: 23-May-2025
lastUpdated: 23-May-2025
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: ["playwright", "typescript", "lambdatest", "cloud", "scaling", "parallel"]
category: Docker & Grid Scaling
categories: ["Docker & Grid Scaling", "UI Automation", "Playwright", "TypeScript", "Infrastructure"]
excerpt: >-
  Deploy Playwright tests to the LambdaTest cloud using CDP WebSockets. Learn how to configure `LT:Options` and execute advanced SmartUI visual regression commands.
readTime: 4 min read
---

**LambdaTest** is another powerhouse in the cloud execution space, offering blazing-fast execution speeds and one of the best analytical dashboards on the market.

Just like BrowserStack and Sauce Labs, LambdaTest natively supports Playwright's Chrome DevTools Protocol (CDP) WebSocket architecture. This means you do not need to rewrite any of your core `.spec.ts` files to run them in the LambdaTest cloud!

### 1. The LambdaTest Configuration

LambdaTest requires a slightly different capability object format, specifically using the `'LT:Options'` namespace. We can create a dedicated config file to handle this translation.

**File:** `lambdatest.config.ts`

```typescript
import { defineConfig } from '@playwright/test';
 
// Helper function to build the LambdaTest CDP Endpoint
const getLambdaTestEndpoint = (browserName: string, browserVersion: string, platform: string) => {
  const capabilities = {
    browserName,
    browserVersion,
    'LT:Options': {
      platform,
      name: 'Playwright E2E Suite',
      build: `plw-build-${process.env.GITHUB_RUN_ID || 'local'}`,
      user: process.env.LT_USERNAME,
      accessKey: process.env.LT_ACCESS_KEY,
      network: true, // Capture network logs natively in LambdaTest
      video: true,   // Capture video recordings natively in LambdaTest
      console: true, // Capture browser console logs
    }
  };
 
  return `wss://cdp.lambdatest.com/playwright?capabilities=${encodeURIComponent(JSON.stringify(capabilities))}`;
};
 
export default defineConfig({
  testDir: './tests',
  workers: 10, // Adjust this to match your LambdaTest enterprise license limits
  
  projects: [
    {
      name: 'lambdatest-chrome-win11',
      use: {
        connectOptions: {
          wsEndpoint: getLambdaTestEndpoint('Chrome', 'latest', 'Windows 11'),
        },
      },
    },
    {
      name: 'lambdatest-edge-mac',
      use: {
        connectOptions: {
          wsEndpoint: getLambdaTestEndpoint('MicrosoftEdge', 'latest', 'MacOS Ventura'),
        },
      },
    },
  ],
});
```

### 2. Execution

To trigger the test suite, provide your LambdaTest credentials as environment variables and point the CLI to your new config file:

```bash
LT_USERNAME=my_user LT_ACCESS_KEY=my_key npx playwright test --config=lambdatest.config.ts
```

### 3. Advanced Feature: SmartUI

One of the main reasons enterprises choose LambdaTest is its deep integration with **SmartUI**, a visual regression testing engine. 

While Playwright has built-in visual comparisons (`expect(page).toHaveScreenshot()`), SmartUI offloads the image processing to LambdaTest servers. You can capture screenshots using a specific custom CDP command inside your test:

```typescript
test('Visual Regression on LambdaTest', async ({ page }) => {
  await page.goto('/dashboard');
  
  // This sends a custom command over the WebSocket to trigger a SmartUI snapshot
  await page.evaluate(() => {}, `lambdatest_action: ${JSON.stringify({
    action: "smartui.takeScreenshot",
    arguments: { screenshotName: "dashboard-page" }
  })}`);
});
```

### Summary

LambdaTest provides a highly scalable cloud grid that integrates flawlessly with Playwright. By configuring pure CDP WebSockets and leveraging features like native Network Logs, Video Recording, and SmartUI Visual Regression, you can dramatically accelerate both test execution and test debugging across your entire CI/CD pipeline!
