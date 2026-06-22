---
title: "Handling Multiple Windows and Tabs"
date: "2025-02-28"
description: "Modern applications often open links in new tabs. Discover how Playwright makes context switching between multiple browser windows and tabs incredibly easy."
tags: ["Playwright", "TypeScript", "Tabs", "Windows", "Context"]
---

Welcome to Blog 16 of the **Playwright TypeScript Mastery Series**!

In Selenium, switching tabs required storing "Window Handles", looping through sets of strings, and executing `driver.switchTo().window()`. It was messy and prone to timing issues if the tab took too long to open.

Playwright completely revolutionizes this with the `context.waitForEvent('page')` API.

### The Playwright Approach to Tabs

In Playwright, a Browser Context (`context`) manages all of your open pages. When you click a link that opens a new tab (like `<a target="_blank">`), Playwright automatically adds that new page to your context.

To interact with it, you simply tell Playwright: "Wait for a new page to spawn, and save it to a variable!"

Let's test this against our live practice site (`https://practice.mycodeyatra.com/#/frames`):

Create `tests/blog16_windows_tabs.spec.ts`:

```typescript
import { test, expect } from '@playwright/test';
// Notice we are passing 'context' into our test fixture!
test('Switching to a newly opened tab', async ({ context, page }) => {
  await page.goto('https://practice.mycodeyatra.com/#/frames');
  
  // 1. We must wrap our click action in a Promise that waits for the new page event.
  // The context.waitForEvent('page') tells Playwright "Wait for a new tab to spawn"
  const [newPage] = await Promise.all([
    context.waitForEvent('page'),
    page.getByTestId('open-tab-btn').click() // This triggers the new tab
  ]);
  
  // 2. We now have a reference to the 'newPage'!
  // Let's assert something on the NEW tab to prove we switched context.
  // Notice we use newPage.locator, NOT page.locator!
  const successHeader = newPage.locator('h1');
  await expect(successHeader).toHaveText('Success!');
  
  // 3. Print the title of both pages to prove they are different
  console.log(`Original Tab Title: ${await page.title()}`);
  console.log(`New Tab Title: ${await newPage.title()}`);
  
  // 4. Close the new tab and bring focus back to the original page
  await newPage.close();
  
  // Verify the original page is still active
  const windowCount = page.getByTestId('window-count');
  await expect(windowCount).toHaveText('1');
});
```

### Execution Output

When you run `npx playwright test tests/blog16_windows_tabs.spec.ts`:

```
Running 1 test using 1 worker

Original Tab Title: MyCodeYatra | Test Automation Sandbox
New Tab Title: Practice Tab
  OK   1 tests/blog16_windows_tabs.spec.ts:4:7 > Blog 16: Handling Multiple Windows and Tabs > Switching to a newly opened tab (779ms)

  1 passed (2.0s)
```

### Why use `Promise.all()`?

You might wonder why we wrapped the event listener and the click inside a `Promise.all()`. 

If you triggered the click *before* setting up the listener, the tab might open so fast that Playwright misses the event! `Promise.all()` executes both commands simultaneously, ensuring Playwright is actively listening the exact millisecond the click happens.

### Conclusion

Handling new tabs is as simple as catching the new page event and assigning it to a variable. You don't have to switch contexts manually; `page` controls your original tab, and `newPage` controls your second tab!

In **Blog 17**, we will tackle the other half of this topic: **Working with iFrames**!
