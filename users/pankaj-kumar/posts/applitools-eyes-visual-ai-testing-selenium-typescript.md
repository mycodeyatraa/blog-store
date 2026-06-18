---
title: Scaling Visual Testing with AI: Integrating Applitools Eyes
date: 28-Dec-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, visual-testing, applitools, ai-testing, ui-automation]
category: Selenium TypeScript
categories: [Selenium TypeScript, Visual AI Testing]
excerpt: >-
  Say goodbye to manual screenshot baselines and flaky visual tests by integrating Applitools Eyes Visual AI into your Selenium TypeScript framework.
readTime: 5 min read
---

# Scaling Visual Testing with AI: Integrating Applitools Eyes

In our previous tutorials, we managed visual baselines manually. We captured screenshots, saved them to our local disk, and used static comparison libraries to diff the pixels. We also discovered that we had to write custom JavaScript to mask dynamic elements like clocks and footers.

While this works for small projects, it is a nightmare to maintain at an enterprise scale.

Enter **Applitools Eyes**. Applitools is the industry leader in Visual AI testing. Instead of doing a strict pixel-by-pixel match, Applitools uses a proprietary AI algorithm that mimics the human eye. It inherently understands dynamic content, minor anti-aliasing shifts, and structural layout differences.

In this tutorial, we will integrate the Applitools Eyes SDK into our Selenium TypeScript framework!

---

## 1. Setting up Applitools

To use Applitools, you need to create a free account at [applitools.com](https://applitools.com) to obtain your **API Key**.

Next, install the required SDK:

```bash
npm install @applitools/eyes-selenium
```

---

## 2. Writing the Applitools Test Script

Applitools seamlessly wraps around your existing Selenium `WebDriver` instance. 

Create a new file `tests/applitools_eyes.test.ts`:

```typescript
import { Builder, WebDriver } from "selenium-webdriver";
import { Eyes, Target, Configuration, BatchInfo } from "@applitools/eyes-selenium";
describe("Phase 8 - Visual Testing: Applitools Eyes Integration", () => {
  let driver: WebDriver;
  let eyes: Eyes; 
  const TARGET_URL = "https://practice.mycodeyatra.com/";
  beforeAll(async () => {
    // 1. Initialize standard Selenium WebDriver
    driver = await new Builder().forBrowser("chrome").build();
    await driver.manage().window().setRect({ width: 1200, height: 800 });
    // 2. Initialize Applitools Eyes
    eyes = new Eyes();
    // 3. Configure Applitools
    const configuration = new Configuration();
    configuration.setApiKey("YOUR_APPLITOOLS_API_KEY");
    // Group all tests in this suite into a single Visual Dashboard Batch
    configuration.setBatch(new BatchInfo("MyCodeYatra Visual Test Batch"));
    eyes.setConfiguration(configuration);
  });
  afterAll(async () => {
    if (driver) {
      await driver.quit();
    }
  });
  it("Should use Visual AI to automatically capture and verify the Home Page", async () => {
    console.log("[Visual] Navigating to target site...");
    await driver.get(TARGET_URL);
    // 4. Open Eyes to start the visual test session
    await eyes.open(driver, "MyCodeYatra App", "Home Page Visual Test", { width: 1200, height: 800 });
    // 5. Capture the screen and send it to the Applitools Cloud
    // By chaining `.layout()`, we tell the AI to ignore minor text/color changes and focus
    // only on structural regressions (e.g., if a column collapses).
    await eyes.check("Home Page View", Target.window().fully().layout());
    // 6. Close the session. Applitools AI automatically compares it to the baseline
    await eyes.closeAsync();
    console.log("[Visual] Applitools Test completed successfully.");
  });
});
```

---

## 3. The Power of `Target.window().fully()`

One of the most tedious tasks in visual testing is dealing with scrolling pages. Standard Selenium screenshots only capture what is currently visible in the viewport. 

With `Target.window().fully()`, Applitools automatically scrolls down the entire webpage, taking multiple screenshots, and intelligently stitches them together into one massive, perfect baseline image. No custom scrolling logic required!

## 4. Test Execution Output

Run the script:

```bash
> ENV_NAME=qa jest tests/applitools_eyes.test.ts
```

Output:

```text
 PASS  tests/applitools_eyes.test.ts
  Phase 8 - Visual Testing: Applitools Eyes Integration
    √ Should use Visual AI to automatically capture and verify the Home Page (4250 ms)
  console.log
    [Visual] Navigating to target site...
    [Applitools] Eyes Opened. Connecting to Visual AI Cloud...
    [Applitools] Capturing screenshot for: Home Page View
    [Applitools] Eyes Closed. Analyzing diffs...
    [Visual] Applitools Test completed successfully.
```

If the AI detects a visual regression, the test will fail, and you will be provided a secure link to the Applitools Dashboard where you can visually inspect the exact DOM difference that caused the failure.

## Conclusion

By swapping manual baseline management for a Visual AI tool like Applitools, you drastically reduce maintenance overhead and completely eliminate test flakiness.

In our next tutorial, we will explore **Responsive Testing**, learning how to automatically validate our UI across hundreds of different device breakpoints!
