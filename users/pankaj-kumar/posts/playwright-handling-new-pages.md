---
title: "Handling New Pages (context.on('page'))"
date: "2025-04-09"
description: "Learn how to capture and control new tabs and pop-up windows opened automatically by user actions in Playwright TypeScript."
tags: ["Playwright", "TypeScript", "Pages", "Tabs", "Events"]
---

Welcome to Blog 57 of the **Playwright TypeScript Mastery Series**!

In web applications, clicking an external link, a button, or selecting a menu option often launches a new browser tab or pop-up window. Because this is asynchronous and event-driven, programmatically grabbing the reference to the newly opened tab requires a reliable pattern.

Today, we'll see how to capture these pages using the browser context's `page` event.

---

### Understanding the Event Pattern

Instead of opening a page manually, we wait for a click or script execution to trigger the window opening. 

To prevent race conditions (where the tab opens before we start listening, or we listen indefinitely for something that failed to open), we use `context.waitForEvent('page')`.

---

### Step 1: Implementing the New Page Event Spec

Create a new test file `tests/blog57_new_page_event.spec.ts`:

```typescript
import { test, expect } from '@playwright/test';
 
test.describe('Blog 57: Handling New Pages via context.on(page)', () => {
 
  test('Capture a new tab/window triggered by user action', async ({ context, page }) => {
    // 1. Navigate to a starting page
    await page.goto('https://example.com');
 
    // 2. Start waiting for the 'page' event before triggering the action
    const pagePromise = context.waitForEvent('page');
 
    // 3. Trigger the window opening via page.evaluate
    await page.evaluate(() => {
      window.open('https://playwright.dev', '_blank');
    });
 
    // 4. Await the new page to open
    const newPage = await pagePromise;
 
    // 5. Verify the new page is loaded and control it
    await newPage.waitForLoadState();
    await expect(newPage).toHaveTitle(/Playwright/);
 
    console.log(`Successfully intercepted new page: ${newPage.url()}`);
    expect(newPage.url()).toContain('playwright.dev');
 
    // Clean up
    await newPage.close();
  });
 
});
```

---

### Step 2: Running the Test

Execute the test in your workspace:

```
npx playwright test tests/blog57_new_page_event.spec.ts
```

**Output:**

```
Running 1 test using 1 worker
 
Successfully intercepted new page: https://playwright.dev/
  ✓  1 tests/blog57_new_page_event.spec.ts:5:7 › Blog 57: Handling New Pages via context.on(page) › Capture a new tab/window triggered by user action (824ms)
 
  1 passed (2.5s)
```

In the next blog, we will learn how to automate **Working with Dialogs (Alerts, Confirms, Prompts)**!
