---
title: Beyond the DOM: An Introduction to Visual Testing
date: 26-Dec-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, visual-testing, screenshot, pixelmatch, css-regression]
category: Selenium TypeScript
categories: [Selenium TypeScript, Visual AI Testing]
excerpt: >-
  Step into Phase 8 of Selenium automation by learning how to capture full-page and component-level screenshot baselines to catch visual CSS regressions.
readTime: 4 min read
---

# Beyond the DOM: An Introduction to Visual Testing

Welcome to **Phase 8**, the final frontier of modern UI Automation: **Visual Testing**!

Up until now, our Selenium tests have relied exclusively on the Document Object Model (DOM). We assert that an element exists, contains specific text, or is clickable. 

But what happens if a developer accidentally changes a CSS file, causing a massive blue banner to obscure the "Submit" button? 
* The button still exists in the DOM.
* The text is correct.
* The test passes.
* **But the user cannot click the button!**

Functional UI testing cannot catch CSS regressions, rendering bugs, or layout shifts. For that, we need Visual Testing.

---

## 1. What is Visual Testing?

Visual Testing (or Visual Regression Testing) works by comparing images rather than HTML tags. 
1. You run a test and capture a **Baseline Image** of the page (or component).
2. The next time the test runs, you capture a **New Image**.
3. A pixel-by-pixel comparison algorithm diffs the two images.
4. If a certain percentage of pixels differ, the test fails, highlighting the exact visual mismatch!

---

## 2. Capturing Baselines with Selenium

Before we use complex comparison algorithms or AI-driven tools, we must first understand how to capture raw image data using Selenium WebDriver.

Selenium provides two powerful methods for capturing images:
* `driver.takeScreenshot()`: Captures the entire visible viewport.
* `element.takeScreenshot()`: Captures *only* the specific web element (e.g., just the navigation bar).

Create `tests/visual_testing.test.ts`:

```typescript
import { Builder, WebDriver, By } from "selenium-webdriver";
import fs from "fs";
import path from "path";
describe("Phase 8 - Visual Testing: Introduction to Image Comparison", () => {
  let driver: WebDriver;
  const TARGET_URL = "https://practice.mycodeyatra.com/";
  beforeAll(async () => {
    driver = await new Builder().forBrowser("chrome").build();
    // CRITICAL: Visual tests must have a strictly defined viewport size!
    await driver.manage().window().setRect({ width: 1920, height: 1080 });
  });
  afterAll(async () => {
    if (driver) {
      await driver.quit();
    }
  });
  it("Should capture a full-page screenshot and save it as a baseline", async () => {
    console.log("[Visual] Navigating to target site...");
    await driver.get(TARGET_URL);
    console.log("[Visual] Taking full-page screenshot...");
    // Returns a Base64 encoded string of the image
    const screenshotBase64 = await driver.takeScreenshot();
    // Save the baseline image to disk
    const screenshotsDir = path.resolve(__dirname, "__image_snapshots__");
    if (!fs.existsSync(screenshotsDir)) {
      fs.mkdirSync(screenshotsDir, { recursive: true });
    }
    const baselinePath = path.join(screenshotsDir, "home_page_baseline.png");
    fs.writeFileSync(baselinePath, screenshotBase64, "base64");
    console.log(`[Visual] Baseline screenshot saved to: ${baselinePath}`);
    expect(fs.existsSync(baselinePath)).toBe(true);
  });
  it("Should capture a specific component (e.g., Header) for localized visual testing", async () => {
    // Component-level visual testing prevents layout changes in the footer from breaking header tests
    const headerElement = await driver.findElement(By.css("header"));
    // Captures only the pixels belonging to this element!
    const elementScreenshot = await headerElement.takeScreenshot();
    const headerBaselinePath = path.resolve(__dirname, "__image_snapshots__", "header_baseline.png");
    fs.writeFileSync(headerBaselinePath, elementScreenshot, "base64");
    console.log(`[Visual] Component screenshot saved to: ${headerBaselinePath}`);
    expect(fs.existsSync(headerBaselinePath)).toBe(true);
  });
});
```

---

## 3. Test Execution Output

Run the visual capture script:

```bash
> ENV_NAME=qa jest tests/visual_testing.test.ts
```

Output:

```text
 PASS  tests/visual_testing.test.ts (6.32 s)
  Phase 8 - Visual Testing: Introduction to Image Comparison
    √ Should capture a full-page screenshot and save it as a baseline (2150 ms)
    √ Should capture a specific component (e.g., Header) for localized visual testing (800 ms)
  console.log
    [Visual] Navigating to target site...
    [Visual] Taking full-page screenshot...
    [Visual] Baseline screenshot saved to: ...\__image_snapshots__\home_page_baseline.png
    [Visual] Component screenshot saved to: ...\__image_snapshots__\header_baseline.png
```

If you look in your `__image_snapshots__` directory, you will now see physical `.png` files of your website!

## Conclusion

We have successfully generated our baseline images. However, manually comparing hundreds of images per release is impossible. 

In our next tutorial, we will explore **Baseline Management**, where we introduce pixel-comparison libraries like `jest-image-snapshot` to automate the diffing process!
