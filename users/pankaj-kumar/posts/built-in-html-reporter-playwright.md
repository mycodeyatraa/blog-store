---
title: Mastering the Playwright Built-in HTML Reporter
date: 08-May-2025
lastUpdated: 08-May-2025
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: ["playwright", "typescript", "reporting", "html", "trace-viewer", "debugging"]
category: Reporting
categories: ["Reporting", "UI Automation", "Playwright", "TypeScript", "Debugging"]
excerpt: >-
  Unlock the power of Playwright's native HTML reporter to automatically capture videos, screenshots, and the time-traveling Trace Viewer for zero-dependency debugging.
readTime: 4 min read
---

While third-party tools like Allure are great for historical dashboards, Playwright comes with a **Built-in HTML Reporter** that is arguably the best single-run debugging tool in the industry.

The built-in HTML reporter is a completely standalone zero-dependency webpage. It automatically embeds your test code, console logs, network requests, screenshots, full video recordings, and the incredibly powerful **Playwright Trace Viewer** right into the browser!

### 1. Configuration

The HTML reporter is usually enabled by default, but you can customize it inside `playwright.config.ts` to ensure it captures artifacts automatically when a test fails.

**File:** `playwright.config.ts`

```typescript
import { defineConfig } from '@playwright/test';
 
export default defineConfig({
  // 1. Enable the HTML reporter
  reporter: [['html', { open: 'never', outputFolder: 'playwright-report' }]],
  
  use: {
    // 2. Automatically capture artifacts on failure!
    screenshot: 'only-on-failure',
    video: 'retain-on-failure',
    trace: 'retain-on-failure',
  },
});
```

### 2. Custom Annotations and Attachments

You can enrich the HTML report directly from your `.spec.ts` files using the `test.info()` object. This is highly useful for attaching manual screenshots or adding links to external issue trackers.

**File:** `tests/html-reporter.spec.ts`

```typescript
import { test, expect } from '@playwright/test';
 
test('Successful login test with manual annotations', async ({ page }) => {
  // 1. Add custom annotations to the top of the HTML report
  test.info().annotations.push({ 
    type: 'issue', 
    description: 'https://github.com/my-repo/issues/42' 
  });
  test.info().annotations.push({ 
    type: 'environment', 
    description: 'Production' 
  });
 
  await page.goto('/login');
  
  // 2. Attach a manual screenshot to the HTML report explicitly
  const screenshot = await page.screenshot();
  await test.info().attach('manual-screenshot', {
    body: screenshot,
    contentType: 'image/png',
  });
});
```

### 3. Viewing the Trace Viewer

When a test fails and `trace: 'retain-on-failure'` is enabled, Playwright generates a `.zip` file containing a full DOM snapshot of every single action, every network request, and every console log that happened during the test.

This zip file is natively embedded into the HTML Report!

```bash
# Run the tests
npx playwright test
 
# Serve the HTML report locally
npx playwright show-report
```

When you open the report and click on a failed test, you will see a **"Traces"** section at the bottom. Clicking the trace image opens a time-travel debugger where you can inspect the exact DOM state before, during, and after the failure occurred.

### Summary

The Playwright Built-in HTML Reporter is your first line of defense for debugging. By configuring it to capture Screenshots, Videos, and Traces `on-failure`, you instantly eliminate the "it works on my machine" problem—you can literally watch a video of the failure and inspect the DOM from your CI pipeline!
