---
title: Navigating Multiple Windows & Browser Tabs in Selenium TypeScript
date: 12-Nov-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, tabs, windows, window-handles]
category: Selenium TypeScript
categories: [Selenium TypeScript, UI Automation, Core Automation]
excerpt: >-
  When you click a link that opens a new tab, your Selenium script will crash if it tries to find elements. Learn how to switch contexts using Window Handles.
readTime: 6 min read
---

# Navigating Multiple Windows & Browser Tabs in Selenium TypeScript

Have you ever clicked a link, a new tab opened, and suddenly your Selenium script crashed because it couldn't find an element?

By default, Selenium does not automatically switch focus to new tabs or popups. The WebDriver object remains stubbornly attached to the **original window** until you explicitly command it to switch contexts.

In this tutorial, we will learn how to master Window Handles, iterate through open tabs, and switch contexts back and forth using TypeScript on **[practice.mycodeyatra.com/#/frames](https://practice.mycodeyatra.com/#/frames)**.

---

## 1. Understanding Window Handles

Every open tab or window in Chrome has a unique alphanumeric ID assigned by the browser, called a **Window Handle**.

To switch to a new tab, you must fetch all currently open handles, find the one that doesn't match your original window, and call `.switchTo().window(handle)`.

```typescript
// 1. Save the original window ID so we can return later
const originalWindow = await driver.getWindowHandle();
// 2. Fetch an array of all open window IDs
const allWindows = await driver.getAllWindowHandles();
```

---

## 2. Switching to the New Tab

When you click a link that opens a new tab, it takes a fraction of a second for the browser to spawn it. Always use an Explicit Wait to ensure the window count has increased before you try to fetch the handles!

```typescript
// Wait for the new tab to open (Total windows should equal 2)
await driver.wait(
  async () => (await driver.getAllWindowHandles()).length === 2,
  5000
);
// Switch to the new tab
for (const handle of await driver.getAllWindowHandles()) {
  if (handle !== originalWindow) {
    await driver.switchTo().window(handle);
    break;
  }
}
```

---

## 3. Writing the Multiple Windows Test

Let's write a complete Jest test. We will navigate to the Sandbox, click a button to spawn a new tab, switch into it, verify its header text, close it, and safely return to the parent window.

Create `tests/windows.test.ts`:

```typescript
import { Builder, WebDriver, By, until } from "selenium-webdriver";
import "chromedriver";
describe("Core UI Automation - Multiple Windows and Tabs", () => {
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
  it("Should switch between multiple windows and tabs", async () => {
    console.log("Navigating to Frames & Windows Page...");
    await driver.get("https://practice.mycodeyatra.com/#/frames");
    // 1. Get the original window handle
    const originalWindow = await driver.getWindowHandle();
    console.log(`Original Window Handle: ${originalWindow}`);
    // Wait for the open tab button to be located
    await driver.wait(until.elementLocated(By.css("[data-testid='open-tab-btn']")), 5000);
    const openTabBtn = await driver.findElement(By.css("[data-testid='open-tab-btn']"));
    // 2. Click the button to open a new tab
    await openTabBtn.click();
    console.log("Clicked 'Open New Tab' button");
    // 3. Wait for the new window to appear
    await driver.wait(
      async () => (await driver.getAllWindowHandles()).length === 2,
      5000
    );
    // 4. Switch to the new tab
    const allWindows = await driver.getAllWindowHandles();
    for (const windowHandle of allWindows) {
      if (windowHandle !== originalWindow) {
        await driver.switchTo().window(windowHandle);
        console.log("Switched to New Tab!");
        break;
      }
    }
    // 5. Interact with elements in the new tab
    const newTabHeader = await driver.wait(until.elementLocated(By.css("h1")), 5000);
    expect(await newTabHeader.getText()).toEqual("Success!");
    console.log("Verified 'Success!' text inside the new tab.");
    // Close the new tab
    await driver.close();
    console.log("Closed the new tab");
    // 6. Switch back to the original window
    await driver.switchTo().window(originalWindow);
    console.log("Switched back to Original Window");
    // Verify we are back by checking the window count UI on the parent
    const windowCount = await driver.findElement(By.css("[data-testid='window-count']"));
    expect(await windowCount.getText()).toEqual("1");
    console.log("Successfully returned to the parent window and verified state.");
  });
});
```

---

## 4. Test Execution Output

```text
> mcyt-sel-typescript@1.0.0 test
> jest
 PASS  tests/windows.test.ts (7.214 s)
  Core UI Automation - Multiple Windows and Tabs
    √ Should switch between multiple windows and tabs (3612 ms)
  console.log
    Navigating to Frames & Windows Page...
  console.log
    Original Window Handle: 0725FE8196E63A75B43B2A63D3BD01F2
  console.log
    Clicked 'Open New Tab' button
  console.log
    Switched to New Tab!
  console.log
    Verified 'Success!' text inside the new tab.
  console.log
    Closed the new tab
  console.log
    Switched back to Original Window
  console.log
    Successfully returned to the parent window and verified state.
Test Suites: 1 passed, 1 total
Tests:       1 passed, 1 total
Snapshots:   0 total
Time:        7.514 s
Ran all test suites.
```

## Conclusion

Switching between multiple windows and browser tabs is a fundamental skill in Enterprise UI automation. Always remember:
1. Save your parent window handle before clicking a link.
2. Explicitly wait for `driver.getAllWindowHandles().length` to increase.
3. Close the popup, then immediately `driver.switchTo().window(originalWindow)`.

In our next tutorial, we will conquer another context-switching nightmare: **Interacting with nested HTML iFrames!**
