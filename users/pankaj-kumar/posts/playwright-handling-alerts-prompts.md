---
title: "Handling Alerts, Prompts, and Dialogs"
date: "2025-02-26"
description: "Web browsers throw native popups that can block your automation. Learn how Playwright seamlessly dismisses or accepts JavaScript alerts, prompts, and confirmation dialogs."
tags: ["Playwright", "TypeScript", "Alerts", "Modals", "Dialogs"]
---

Welcome to Blog 14 of the **Playwright TypeScript Mastery Series**!

We have reached the final piece of our Form Interaction deep dive. Sometimes, an application will throw a popup at you. 

There are two completely different types of popups on the web:
1. **Native Browser Dialogs**: These are triggered by JavaScript (`alert()`, `confirm()`, `prompt()`). They physically block the browser, and you cannot inspect them using Chrome DevTools!
2. **Custom Web Modals**: These are built using HTML/CSS (like Bootstrap Modals) and are standard parts of the DOM.

Let's master both!

### 1. Native JavaScript Alerts

By default, **Playwright automatically dismisses (cancels) all native dialogs!** 

If your test clicks a button that triggers an `alert()`, Playwright instantly closes it so your test doesn't freeze. 

But what if you *want* to interact with it? What if you want to click "OK" or read the text inside the alert? You must setup a **Dialog Listener** *before* you click the button!

Create `tests/blog14_alerts.spec.ts` using our live practice sandbox (`https://practice.mycodeyatra.com/#/overlays`):

```typescript
import { test, expect } from '@playwright/test';
test('Handling a JavaScript Alert Dialog', async ({ page }) => {
  await page.goto('https://practice.mycodeyatra.com/#/overlays');
  
  // 1. Setup the Dialog Listener BEFORE triggering the alert!
  page.on('dialog', async dialog => {
    // Print the alert message to the console
    console.log(`Alert Message: ${dialog.message()}`);
    // Accept (Click OK) on the alert
    await dialog.accept();
    
    // Note: Use await dialog.dismiss() if you want to click Cancel
    // Use await dialog.accept('John') if dealing with a prompt()
  });

  // 2. Click the button that triggers the JS Alert
  await page.getByTestId('alert-btn').click();
  
  // 3. Verify the web page registered our 'accept' action
  await expect(page.getByTestId('alert-result')).toContainText('Alert dismissed');
});
```

### 2. Custom Web Modals

A custom web modal is *not* a native browser dialog. It is just a regular HTML `<div>` floating in the center of the screen. Because it is part of the DOM, you do **not** use `page.on('dialog')`. 

You simply locate the elements like normal!

```typescript
test('Handling a Custom Web Modal', async ({ page }) => {
  await page.goto('https://practice.mycodeyatra.com/#/overlays');
  
  // 1. Trigger the custom DOM modal
  await page.getByTestId('modal-trigger').click();
  
  // 2. The modal is part of the DOM, so we handle it like a normal element!
  const modalContent = page.getByTestId('modal-content');
  await expect(modalContent).toBeVisible();
  
  // 3. Click the close button inside the modal
  await page.getByTestId('modal-close').click();
  
  // 4. Verify the modal is gone
  await expect(modalContent).not.toBeVisible();
});
```

### Execution Output

When you run `npx playwright test tests/blog14_alerts.spec.ts`:

```bash
Running 2 tests using 1 worker

Alert Message: This is a standard JavaScript Alert!
  ✓  1 tests\blog14_alerts.spec.ts:4:7 › Blog 14: Handling Alerts and Modals › Handling a JavaScript Alert Dialog (795ms)
  ✓  2 tests\blog14_alerts.spec.ts:22:7 › Blog 14: Handling Alerts and Modals › Handling a Custom Web Modal (628ms)

  2 passed (2.8s)
```

### Conclusion

You now know how to differentiate and handle native browser dialogs using `page.on('dialog')`, and how to handle modern Custom DOM Modals using standard Playwright Locators!

In **Blog 15**, we are going to dive into the most powerful automation feature of Playwright: **Auto-Waiting and Network Interception**!
