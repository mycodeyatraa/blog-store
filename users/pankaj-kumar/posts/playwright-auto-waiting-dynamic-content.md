---
title: "Auto-Waiting and Dynamic Content"
date: "2025-02-27"
description: "Say goodbye to Thread.sleep()! Learn how Playwright's revolutionary auto-waiting seamlessly handles dynamic loading, progress bars, and network requests."
tags: ["Playwright", "TypeScript", "Waits", "Dynamic", "Synchronization"]
---

Welcome to Blog 15 of the **Playwright TypeScript Mastery Series**!

One of the biggest reasons automation tests fail (or become "flaky") is synchronization. 

In older frameworks like Selenium, if you clicked a button and the web page took 3 seconds to load the next element, your script would instantly fail because it tried to interact with the element before it existed. To fix this, testers wrote horrible, brittle code like `Thread.sleep(5000)`.

Playwright completely solved this problem with **Auto-Waiting**.

### 1. Waiting for Elements to Appear

When you try to perform an action (like `click()`, `fill()`, or `expect().toBeVisible()`), Playwright does **not** fail immediately if the element isn't there. 

Instead, it intelligently polls the DOM. It waits for the element to attach to the DOM, become visible, stop animating, and become capable of receiving events. By default, it will wait up to **5 seconds** before throwing an error.

Let's test this using our live sandbox (`https://practice.mycodeyatra.com/#/dynamic-content`):

Create `tests/blog15_dynamic_content.spec.ts`:

```typescript
import { test, expect } from '@playwright/test';
test('Playwright Auto-Waits for delayed elements', async ({ page }) => {
  await page.goto('https://practice.mycodeyatra.com/#/dynamic-content');
  // 1. Click the button that triggers a slow 3-second network request
  await page.getByTestId('start-loading-btn').click();
  // 2. Try to assert text on an element that DOES NOT EXIST yet!
  // Playwright will NOT fail! It will automatically wait for the 3-second 
  // delay to finish, the element to render, and the text to appear.
  const delayedBox = page.getByTestId('delayed-content');
  await expect(delayedBox).toHaveText('Content loaded after 3 seconds!');
});
```

### 2. Waiting for Elements to become Enabled

Auto-waiting doesn't just look for an element to exist. It waits for it to become **actionable**. 

Imagine a progress bar. At the end of the progress bar is a "Complete" button. The button exists in the DOM, but it has the HTML `disabled` attribute until the progress reaches 100%.

If you tell Playwright to click it, it will automatically wait until the `disabled` attribute is removed!

```typescript
test('Playwright Auto-Waits for actionability', async ({ page }) => {
  await page.goto('https://practice.mycodeyatra.com/#/dynamic-content');
  // 1. Start the progress bar (takes about 5 seconds to finish)
  await page.getByTestId('start-progress-btn').click();
  // Dismiss the alert that pops up when we click the final button
  page.on('dialog', dialog => dialog.accept());
  // 2. Try to click the "Click when 100%" button immediately!
  // The button is DISABLED. Playwright intelligently WAITS until 
  // the button's 'disabled' attribute is removed before clicking!
  const completeBtn = page.getByTestId('progress-complete-btn');
  await completeBtn.click(); 
  
  console.log('Successfully clicked the button after it became enabled!');
});
```

### Execution Output

When you run `npx playwright test tests/blog15_dynamic_content.spec.ts`:

```
Running 2 tests using 1 worker
 
  OK   1 tests/blog15_dynamic_content.spec.ts:4:7 > Blog 15: Auto-Waiting and Dynamic Content > Playwright Auto-Waits for delayed elements (4.1s)
Successfully clicked the button after it became enabled!
  OK   2 tests/blog15_dynamic_content.spec.ts:17:7 > Blog 15: Auto-Waiting and Dynamic Content > Playwright Auto-Waits for actionability (enabled state) (5.8s)
 
  2 passed (11.3s)
```

Notice the execution time! Test 1 took `4.1s` (waiting for the 3s delay) and Test 2 took `5.8s` (waiting for the progress bar). We didn't write a *single* `sleep()` command!

### Conclusion

Playwright's auto-waiting is its "killer feature". By relying on Playwright's built-in actionability checks, you can completely eliminate flaky tests without writing complex synchronization logic.

In **Blog 16**, we will tackle another major headache for automation engineers: **Handling Multiple Browser Windows and Tabs**!
