---
title: Mastering Frames and iFrames in Selenium TypeScript
date: 13-Nov-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, frames, iframes, context-switching]
category: Selenium TypeScript
categories: [Selenium TypeScript, UI Automation, Core Automation]
excerpt: >-
  If an element is inside an iframe, Selenium is completely blind to it until you explicitly switch contexts! Learn how to safely navigate iFrames using TypeScript.
readTime: 6 min read
---

# Mastering Frames and iFrames in Selenium TypeScript

One of the most common reasons a `NoSuchElementError` occurs is because the target element is trapped inside an `<iframe>`.

An `<iframe>` is essentially a webpage embedded inside another webpage. Selenium can only "see" elements that belong to the active DOM context. If an element lives inside an iframe, Selenium is completely blind to it until you explicitly switch contexts!

In this tutorial, we will learn how to detect iframes, switch contexts into them, and switch back out using TypeScript on **[practice.mycodeyatra.com/#/frames](https://practice.mycodeyatra.com/#/frames)**.

---

## 1. How to Switch into an iFrame

You can switch into an iframe in three ways:
1. By **Index** (e.g., `driver.switchTo().frame(0)`) - Not recommended, breaks if page structure changes.
2. By **Name or ID** string.
3. By passing the actual **WebElement** of the iframe. This is the safest and most robust approach in Enterprise frameworks.

```typescript
// 1. Locate the iframe element
const iframeElement = await driver.findElement(By.css("[data-testid='iframe-container']"));
// 2. Switch context
await driver.switchTo().frame(iframeElement);
// Now Selenium can "see" elements inside the iframe!
await driver.findElement(By.css("#iframe-btn")).click();
```

---

## 2. Returning to the Parent Frame

Once you switch into an iframe, Selenium is blind to the outer (parent) webpage! 
If you try to click a navigation menu outside the iframe, your test will fail.

You **must** switch back to the main document context when you are done interacting with the iframe:

```typescript
// Switch context back to the Default Parent Content
await driver.switchTo().defaultContent();
```

*(Note: If you have deeply nested iframes, you can use `await driver.switchTo().parentFrame()` to move up exactly one level.)*

---

## 3. Writing the iFrames Test

Let's write a complete Jest test. We will navigate to the Sandbox, locate the iframe container, switch inside it, click the hidden button, verify the resulting text, and finally switch back out to verify a parent element.

Create `tests/iframes.test.ts`:

```typescript
import { Builder, WebDriver, By, until } from "selenium-webdriver";
import "chromedriver";
describe("Core UI Automation - Frames and iFrames", () => {
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
  it("Should switch into an iFrame and interact with nested elements", async () => {
    console.log("Navigating to Frames & Windows Page...");
    await driver.get("https://practice.mycodeyatra.com/#/frames");
    // Wait for the iframe container to be located
    const iframeElement = await driver.wait(
      until.elementLocated(By.css("[data-testid='iframe-container']")), 
      5000
    );
    // 1. Switch context to the iFrame
    await driver.switchTo().frame(iframeElement);
    console.log("Switched context into the iFrame successfully!");
    // 2. Interact with elements INSIDE the iFrame
    const iframeBtn = await driver.findElement(By.css("#iframe-btn"));
    await iframeBtn.click();
    console.log("Clicked the button inside the iFrame");
    // Verify the message that appears inside the iFrame
    const msg = await driver.findElement(By.css("#msg"));
    expect(await msg.getText()).toEqual("IFrame Button Clicked!");
    console.log("Verified text inside the iFrame.");
    // 3. Switch context back to the Default (Parent) Content
    await driver.switchTo().defaultContent();
    console.log("Switched context back to the default parent page.");
    // Verify we are back by finding an element outside the iFrame
    const parentHeader = await driver.findElement(By.css("h2"));
    expect(await parentHeader.getText()).toContain("iFrames & Windows");
    console.log("Successfully verified parent DOM interaction!");
  });
});
```

---

## 4. Test Execution Output

```text
> mcyt-sel-typescript@1.0.0 test
> jest
 PASS  tests/iframes.test.ts (6.425 s)
  Core UI Automation - Frames and iFrames
    √ Should switch into an iFrame and interact with nested elements (2104 ms)
  console.log
    Navigating to Frames & Windows Page...
  console.log
    Switched context into the iFrame successfully!
  console.log
    Clicked the button inside the iFrame
  console.log
    Verified text inside the iFrame.
  console.log
    Switched context back to the default parent page.
  console.log
    Successfully verified parent DOM interaction!
Test Suites: 1 passed, 1 total
Tests:       1 passed, 1 total
Snapshots:   0 total
Time:        6.650 s
Ran all test suites.
```

## Conclusion

Whenever you encounter a `NoSuchElementError` for an element you *know* is visible on the screen, always open Chrome DevTools and check if it is wrapped inside an `<iframe>` tag.

Switching frames is a critical skill for automating embedded widgets, third-party payment gateways, and rich-text editors. 

In our next tutorial, we will tackle the modern equivalent of iframes: **Bypassing the Shadow DOM!**
