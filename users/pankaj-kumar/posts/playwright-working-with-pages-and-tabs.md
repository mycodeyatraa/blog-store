---
title: "Working with Pages and Tabs"
date: "2025-04-08"
description: "Learn how to open, coordinate, and control multiple browser tabs and pages inside a single Browser Context in Playwright TypeScript."
tags: ["Playwright", "TypeScript", "Pages", "Tabs", "newPage"]
---

Welcome to Blog 56 of the **Playwright TypeScript Mastery Series**!

In web automation, you often need to deal with multi-tab or multi-window workflows. For example, clicking an external link might open a new tab, or you might need to open a secondary tab to view admin configurations while executing tasks as a regular user on the primary tab.

Today, we will learn how to open and coordinate multiple pages (or tabs) within a single `BrowserContext`.

---

### Key Concepts: Context vs. Page

- **Browser Context**: Acts as an isolated browser session (like an Incognito window). It contains its own cache, storage, and cookies.
- **Page (Tab)**: A single tab or window within that browser context. Multiple pages in the same context share storage, state, and cookies.

---

### Step 1: Implementing the Pages and Tabs Spec

Create a new test file `tests/blog56_tabs.spec.ts`:

```typescript
import { test, expect } from '@playwright/test';

test.describe('Blog 56: Working with Pages and Tabs', () => {

  test('Open and manage multiple pages/tabs programmatically', async ({ context, page }) => {
    // 1. Navigate page 1 (the default tab) to example.com
    await page.goto('https://example.com');
    await expect(page).toHaveTitle(/Example Domain/);

    // 2. Open a second tab/page within the same browser context
    const newPage = await context.newPage();
    await newPage.goto('https://playwright.dev');
    await expect(newPage).toHaveTitle(/Playwright/);

    // 3. Make sure we can switch operations between pages easily
    console.log(`Tab 1 URL: ${page.url()}`);
    console.log(`Tab 2 URL: ${newPage.url()}`);

    expect(page.url()).toContain('example.com');
    expect(newPage.url()).toContain('playwright.dev');

    // 4. Close the secondary tab
    await newPage.close();

    // Verify default page is still open and responsive
    await expect(page).toHaveTitle(/Example Domain/);
  });

});
```

---

### Step 2: Running the Test

Run the test in your workspace:

```
npx playwright test tests/blog56_tabs.spec.ts
```

**Output:**

```
Running 1 test using 1 worker

Tab 1 URL: https://example.com/
Tab 2 URL: https://playwright.dev/
  ✓  1 tests/blog56_tabs.spec.ts:5:7 › Blog 56: Working with Pages and Tabs › Open and manage multiple pages/tabs programmatically (874ms)

  1 passed (2.3s)
```

In the next blog, we will learn how to handle asynchronous event-driven tab opening where user actions (like clicking a button) open a new tab, using the **Handling New Pages (context.on('page'))** event listener!
