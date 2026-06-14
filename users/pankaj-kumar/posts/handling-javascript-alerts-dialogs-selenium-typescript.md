---
title: Handling JavaScript Alerts & Dialogs in Selenium TypeScript
date: 10-Nov-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, alerts, dialogs, popups]
category: Selenium TypeScript
categories: [Selenium TypeScript, UI Automation, Core Automation]
excerpt: >-
  When a native browser popup appears, your test will crash unless you handle it properly. Learn how to switch contexts and interact with JS Alerts, Confirms, and Prompts.
readTime: 6 min read
---

# Handling JavaScript Alerts & Dialogs in Selenium TypeScript

Have you ever run an automated test, only to have a JavaScript popup freeze your browser? 
Unlike standard HTML elements, native browser popups (Alerts, Confirms, and Prompts) cannot be inspected using `F12` DevTools. They belong to the browser window itself, not the DOM.

If you try to use `By.css()` to locate a button on an alert, Selenium will throw a `UnhandledAlertException` and crash! 

In this tutorial, we will learn how to properly handle these intrusive popups using the `driver.switchTo().alert()` method in TypeScript on **[practice.mycodeyatra.com/#/overlays](https://practice.mycodeyatra.com/#/overlays)**.

---

## 1. The Three Types of JavaScript Popups

1. **Simple Alert:** A popup with just an "OK" button. Usually used for warnings.
2. **Confirm Dialog:** A popup with "OK" and "Cancel" buttons.
3. **Prompt Dialog:** A popup that asks the user to type text into an input field before clicking OK/Cancel.

---

## 2. Switching Context to the Alert

To interact with an alert, you must explicitly tell Selenium to pause interacting with the HTML DOM and switch its focus to the native browser popup.

**The Golden Rule of Alerts:** Always use an Explicit Wait (`until.alertIsPresent()`) before switching context. If the test runs faster than the browser can render the alert, you will get a `NoAlertPresentException`.

```typescript
// 1. Wait for the alert to appear
await driver.wait(until.alertIsPresent(), 5000);
// 2. Switch context to the alert
const alert = await driver.switchTo().alert();
// 3. Read the text
const text = await alert.getText();
// 4. Accept or Dismiss
await alert.accept();   // Clicks 'OK'
// OR
await alert.dismiss(); // Clicks 'Cancel'
```

---

## 3. Writing the Alerts Test

Let's write a Jest test that clicks buttons to trigger alerts, waits for them, reads their text, and handles them properly.

Create `tests/alerts.test.ts`:

```typescript
import { Builder, WebDriver, By, until } from "selenium-webdriver";
import "chromedriver";
describe("Core UI Automation - Alerts and Dialogs", () => {
  let driver: WebDriver;
  beforeAll(async () => {
    driver = await new Builder().forBrowser("chrome").build();
    await driver.manage().window().maximize();
    await driver.manage().setTimeouts({ implicit: 5000 });
  });
  afterAll(async () => {
    if (driver) {
      await driver.quit();
    }
  });
  it("Should interact with JS Alerts, Confirms, and Prompts", async () => {
    console.log("Navigating to Overlays Page...");
    await driver.get("https://practice.mycodeyatra.com/#/overlays");
    // Wait for page to render
    await driver.wait(until.elementLocated(By.css("[data-testid='alert-btn']")), 5000);
    // ---------------------------------------------------
    // 1. Handling a Simple Alert (OK)
    // ---------------------------------------------------
    const alertBtn = await driver.findElement(By.css("[data-testid='alert-btn']"));
    await alertBtn.click();
    console.log("Clicked Alert Button");
    await driver.wait(until.alertIsPresent(), 5000);
    const alert = await driver.switchTo().alert();
    const alertText = await alert.getText();
    expect(alertText).toContain("This is a standard JavaScript Alert!");
    console.log(`Alert Text: ${alertText}`);
    await alert.accept();
    console.log("Accepted Simple Alert");
    // ---------------------------------------------------
    // 2. Handling a Confirm Dialog (Cancel)
    // ---------------------------------------------------
    const confirmBtn = await driver.findElement(By.css("[data-testid='confirm-btn']"));
    await confirmBtn.click();
    console.log("Clicked Confirm Button");
    await driver.wait(until.alertIsPresent(), 5000);
    const confirmAlert = await driver.switchTo().alert();
    // We use .dismiss() to click the Cancel button
    await confirmAlert.dismiss();
    console.log("Dismissed Confirm Alert");
    // Assert the UI updated correctly after cancellation
    const confirmResult = await driver.wait(until.elementLocated(By.css("[data-testid='alert-result']")), 5000);
    expect(await confirmResult.getText()).toEqual("Result: You clicked Cancel");
  });
});
```

---

## 4. Test Execution Output

```text
> mcyt-sel-typescript@1.0.0 test
> jest
 PASS  tests/alerts.test.ts (6.541 s)
  Core UI Automation - Alerts and Dialogs
    √ Should interact with JS Alerts, Confirms, and Prompts (3254 ms)
  console.log
    Navigating to Overlays Page...
  console.log
    Clicked Alert Button
  console.log
    Alert Text: This is a standard JavaScript Alert!
  console.log
    Accepted Simple Alert
  console.log
    Clicked Confirm Button
  console.log
    Dismissed Confirm Alert
Test Suites: 1 passed, 1 total
Tests:       1 passed, 1 total
Snapshots:   0 total
Time:        6.882 s
Ran all test suites.
```

## Conclusion

Whenever an application triggers a native browser popup, remember that Selenium must step out of the DOM. By using `until.alertIsPresent()` paired with `switchTo().alert()`, you ensure your test framework never crashes abruptly due to unexpected popups.

In our next article, we will cover how to manage complex **File Uploads and Downloads**!
