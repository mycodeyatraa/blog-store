---
title: Faking It Till You Make It: Mobile Web Emulation in Selenium
date: 30-Dec-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, visual-testing, mobile-emulation, cdp, chrome-devtools]
category: Selenium TypeScript
categories: [Selenium TypeScript, Visual AI Testing]
excerpt: >-
  Go beyond viewport resizing by leveraging the Chrome DevTools Protocol to spoof User-Agents, simulate touch events, and render high-DPI retina mobile baselines.
readTime: 4 min read
---

# Faking It Till You Make It: Mobile Web Emulation in Selenium

In our previous tutorial, we achieved Responsive Visual Testing by simply resizing the browser window. While this triggers CSS Media Queries (making the site *look* mobile), it does not simulate a true mobile environment. 

Why does this matter?
1. **User-Agent Sniffing:** Many enterprise servers inspect the incoming `User-Agent` string. If it detects a desktop browser, it might serve an entirely different DOM structure than it would for a mobile browser.
2. **Device Pixel Ratio (DPR):** High-end phones like iPhones have Retina screens. If your visual test captures a standard 1x resolution, it won't match what the mobile user sees!
3. **Touch Events:** Resizing a desktop browser doesn't enable "touch" support. If your UI has swipe-carousels, resizing won't test them properly.

To solve this, we must configure Selenium to tap into the **Chrome DevTools Protocol (CDP)** and activate true **Mobile Web Emulation**.

---

## 1. Configuring ChromeOptions for Emulation

The easiest way to emulate a mobile device in Selenium is to pass a configuration object to `ChromeOptions`. Chrome maintains a hardcoded dictionary of popular devices (like "iPhone 12 Pro", "Pixel 5", etc.).

Create `tests/mobile_emulation.test.ts`:

```typescript
import { Builder, WebDriver } from "selenium-webdriver";
import chrome from "selenium-webdriver/chrome";
import fs from "fs";
import path from "path";
describe("Phase 8 - Visual Testing: Mobile Web Emulation", () => {
  let driver: WebDriver;
  const TARGET_URL = "https://practice.mycodeyatra.com/";
  beforeAll(async () => {
    // 1. Construct ChromeOptions
    const chromeOptions = new chrome.Options();
    // 2. Define the exact device name string you wish to emulate
    // This string must perfectly match a device listed in Chrome DevTools
    const mobileEmulation = {
      deviceName: "iPhone 12 Pro"
    };
    // 3. Attach the emulation config
    chromeOptions.setMobileEmulation(mobileEmulation);
    // 4. Build the driver
    driver = await new Builder()
      .forBrowser("chrome")
      .setChromeOptions(chromeOptions)
      .build();
  });
  afterAll(async () => {
    if (driver) {
      await driver.quit();
    }
  });
  it("Should capture a visual baseline of a fully emulated iPhone environment", async () => {
    console.log("[Emulation] Navigating to target site as an iPhone 12 Pro...");
    // When the browser navigates, it sends an iPhone User-Agent to the server,
    // and the window possesses mobile touch-events and iOS device pixel ratios!
    await driver.get(TARGET_URL);
    // Let's verify that the User-Agent was actually spoofed
    const userAgent = await driver.executeScript("return navigator.userAgent;") as string;
    expect(userAgent).toContain("iPhone");
    console.log(`[Emulation] Verified User-Agent: ${userAgent.substring(0, 50)}...`);
    // Take the screenshot (this will be a high-DPI Retina image!)
    const screenshotBase64 = await driver.takeScreenshot();
    const screenshotsDir = path.resolve(__dirname, "__image_snapshots__");
    if (!fs.existsSync(screenshotsDir)) {
      fs.mkdirSync(screenshotsDir, { recursive: true });
    }
    const baselinePath = path.join(screenshotsDir, "emulated_iphone12_home.png");
    fs.writeFileSync(baselinePath, screenshotBase64, "base64");
    console.log(`[Visual] Captured emulated baseline at: ${baselinePath}`);
    expect(fs.existsSync(baselinePath)).toBe(true);
  });
});
```

---

## 2. Test Execution Output

Run the emulation suite:

```bash
> ENV_NAME=qa jest tests/mobile_emulation.test.ts
```

Output:

```text
 PASS  tests/mobile_emulation.test.ts
  Phase 8 - Visual Testing: Mobile Web Emulation
    √ Should capture a visual baseline of a fully emulated iPhone environment (1850 ms)
  console.log
    [Emulation] Navigating to target site as an iPhone 12 Pro...
    [Emulation] Verified User-Agent: Mozilla/5.0 (iPhone; CPU iPhone OS 13_2_3 like Mac...
    [Visual] Captured emulated baseline at: ...\emulated_iphone12_home.png
```

If you open the generated `.png`, you will notice the image file is actually much larger and sharper than the one generated in our standard Responsive resize test! This is because the DevTools protocol successfully faked the physical pixel density of an iPhone screen!

## Conclusion

Mobile Web Emulation is incredibly powerful, allowing you to catch mobile-specific server-side rendering logic and touch-event UI bugs without needing to pay for a costly real-device farm (like BrowserStack or SauceLabs).

However, what if we want to run our visual tests on *all* browsers (Chrome, Safari, Firefox) without writing a complex Selenium Grid matrix? 

In our final tutorial of Phase 8, we will explore **Percy**, a visual review platform that renders your DOM across every major browser in the cloud!
