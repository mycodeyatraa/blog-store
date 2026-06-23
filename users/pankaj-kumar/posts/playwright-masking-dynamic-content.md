---
title: "Masking Dynamic Content & Retrying Screenshots"
date: "2025-04-04"
description: "Learn how to use masking and element hiding in Playwright TypeScript to exclude dynamic web content (like times, dates, and ads) from visual regression checks."
tags: ["Playwright", "TypeScript", "Visual Testing", "Masking", "Dynamic Content"]
---

Welcome to Blog 52 of the **Playwright TypeScript Mastery Series**!

In real-world web applications, pages often contain dynamic components: timestamps, database-driven user IDs, live pricing tables, or promotional ads. If you capture a screenshot of such a page, the test will fail on subsequent executions because these details change constantly.

Today, we will learn how to use the `mask` option to overlay dynamic elements with solid-color boxes during visual assertions.

---

### Why Masking is Essential

Instead of excluding the entire container or failing the test, Playwright allows you to specify locators to obscure. During screenshot generation:
- The targeted element is covered with a solid magenta/pink box.
- The pixel comparisons ignore whatever was originally displayed underneath that box.
- By default, CSS animations and video playback are also automatically frozen/disabled to avoid transient frames causing failures.

---

### Step 1: Implementing the Masking Spec

Create a test file `tests/blog52_masking.spec.ts` showing how to mask dynamic elements:

```typescript
import { test, expect } from '@playwright/test';
 
test.describe('Blog 52: Masking Dynamic Elements in Visual Testing', () => {
 
  test('Capture screenshot masking dynamic elements', async ({ page }) => {
    // Navigate to a page with dynamic content (e.g. timestamp)
    await page.goto('data:text/html,<html><body>' +
      '<div id="container" style="padding: 20px; background-color: lightgray;">' +
      '  <h2>Dashboard</h2>' +
      '  <div id="timestamp" style="font-family: monospace; margin-top: 10px;">' +
      '    Current Time: ' + new Date().toISOString() +
      '  </div>' +
      '</div>' +
      '</body></html>');
    
    const container = page.locator('#container');
    const timestamp = page.locator('#timestamp');
 
    // Screenshot the container, but mask the dynamic timestamp element
    await expect(container).toHaveScreenshot('dashboard.png', {
      mask: [timestamp],
      animations: 'disabled'
    });
    console.log('[Visual Masking] Container captured and dynamic element masked.');
  });
 
});
```

---

### Step 2: Verification

Run the test. On first execution, run with `--update-snapshots`:

```
npx playwright test tests/blog52_masking.spec.ts --update-snapshots
```

Run it again without the update flag. Even though the timezone/timestamp string changes on each evaluation, the test passes successfully:

```
npx playwright test tests/blog52_masking.spec.ts
```

**Output:**

```
Running 1 test using 1 worker
 
[Visual Masking] Container captured and dynamic element masked.
  ✓  1 tests/blog52_masking.spec.ts:5:7 › Blog 52: Masking Dynamic Elements in Visual Testing › Capture screenshot masking dynamic elements (377ms)
 
  1 passed (1.6s)
```

In the next blog, we will step out of visual testing to explore how to **execute custom JavaScript** inside the browser console using `page.evaluate`!
