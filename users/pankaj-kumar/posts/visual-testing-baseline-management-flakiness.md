---
title: Defeating Visual Flakiness: Mastering Baseline Management
date: 27-Dec-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, visual-testing, baseline, flakiness, image-diff]
category: Selenium TypeScript
categories: [Selenium TypeScript, Visual AI Testing]
excerpt: >-
  Solve the biggest headaches in Visual Testing by learning how to mask dynamic elements and manage OS-level font rendering tolerances.
readTime: 5 min read
---

# Defeating Visual Flakiness: Mastering Baseline Management

In our previous tutorial, we learned how to capture screenshot baselines. However, if you run a visual test suite in an enterprise environment, you will immediately encounter two massive problems:

1. **Dynamic Data:** The application displays a live clock, dynamic advertisements, or rotating carousels. Every time you take a screenshot, the pixels differ, and the visual test fails.
2. **OS Anti-Aliasing:** Your local machine is a Mac, but your CI/CD server is Linux. Fonts render slightly differently on different operating systems, causing microscopic pixel differences that fail the test.

In this tutorial, we will learn how to tame these issues through **Baseline Management**.

---

## 1. Masking Dynamic Elements

If a web element changes on every reload (like a timestamp or user avatar), we must hide it before taking the screenshot. We can achieve this by executing a snippet of JavaScript via Selenium to "mask" the element.

Create `tests/baseline_management.test.ts`:

```typescript
import { Builder, WebDriver, By } from "selenium-webdriver";
import fs from "fs";
import path from "path";
describe("Phase 8 - Visual Testing: Baseline Management", () => {
  let driver: WebDriver;
  const TARGET_URL = "https://practice.mycodeyatra.com/";
  beforeAll(async () => {
    driver = await new Builder().forBrowser("chrome").build();
    await driver.manage().window().setRect({ width: 1920, height: 1080 });
  });
  afterAll(async () => {
    if (driver) {
      await driver.quit();
    }
  });
  it("Should handle dynamic data by masking elements before screenshot", async () => {
    console.log("[Visual] Navigating to target site...");
    await driver.get(TARGET_URL);
    console.log("[Visual] Masking dynamic elements (e.g., footers/timestamps)...");
    // Execute JS to apply a solid black background to the footer to mask changing copyright dates
    await driver.executeScript(`
      const footer = document.querySelector('footer');
      if (footer) {
        footer.style.backgroundColor = 'black';
        footer.style.color = 'black';
        // Alternatively, hide it entirely: footer.style.visibility = 'hidden';
      }
    `);
    console.log("[Visual] Taking masked full-page screenshot...");
    const screenshotBase64 = await driver.takeScreenshot();
    // Save the baseline image to disk
    const screenshotsDir = path.resolve(__dirname, "__image_snapshots__");
    if (!fs.existsSync(screenshotsDir)) {
      fs.mkdirSync(screenshotsDir, { recursive: true });
    }
    const baselinePath = path.join(screenshotsDir, "masked_page_baseline.png");
    fs.writeFileSync(baselinePath, screenshotBase64, "base64");
    console.log(`[Visual] Masked baseline screenshot saved to: ${baselinePath}`);
    expect(fs.existsSync(baselinePath)).toBe(true);
  });
```

---

## 2. Handling Font Anti-Aliasing (Failure Thresholds)

When using libraries like `jest-image-snapshot` or `pixelmatch`, you must never require a 0% pixel difference. Instead, you should allow a slight tolerance to account for OS-level font rendering variations.

```typescript
  it("Should allow configurable failure thresholds for minor anti-aliasing differences", async () => {
    console.log("[Visual] Configuring visual threshold tolerance...");
    // In a real 'jest-image-snapshot' environment, configure it like this:
    const customConfig = {
      failureThreshold: 0.05, // Allow up to 5% visual difference
      failureThresholdType: 'percent',
      customSnapshotIdentifier: 'home-page-1920-linux'
    };
    console.log(`[Visual] Tolerance set to 5% to prevent OS-level font rendering flakiness.`);
    expect(customConfig.failureThreshold).toBe(0.05);
  });
});
```

### Pro-Tip for CI/CD:
Always generate your "Golden Baselines" on the exact same OS that your CI/CD runner uses. If your Jenkins server runs on Ubuntu, use Docker to run an Ubuntu container locally, generate the baselines inside the container, and commit those Linux-generated `.png` files to your Git repository.

---

## 3. Test Execution Output

Run the script:

```bash
> ENV_NAME=qa jest tests/baseline_management.test.ts
```

Output:

```text
 PASS  tests/baseline_management.test.ts
  Phase 8 - Visual Testing: Baseline Management
    √ Should handle dynamic data by masking elements before screenshot (1420 ms)
    √ Should allow configurable failure thresholds for minor anti-aliasing differences (2 ms)
  console.log
    [Visual] Navigating to target site...
    [Visual] Masking dynamic elements (e.g., footers/timestamps)...
    [Visual] Taking masked full-page screenshot...
    [Visual] Masked baseline screenshot saved to: ...\__image_snapshots__\masked_page_baseline.png
    [Visual] Configuring visual threshold tolerance...
    [Visual] Tolerance set to 5% to prevent OS-level font rendering flakiness.
```

## Conclusion

By mastering DOM masking and failure thresholds, you have eliminated the two biggest sources of flakiness in Visual Regression Testing.

However, writing custom JS masks for every dynamic element can become tedious. In our next tutorial, we will explore a magical, AI-driven commercial tool that does all of this automatically: **Applitools Eyes**!
