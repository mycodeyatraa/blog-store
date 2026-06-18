---
title: Breakpoints and CSS: Automating Responsive Visual Tests
date: 29-Dec-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, visual-testing, responsive, css-breakpoints, mobile]
category: Selenium TypeScript
categories: [Selenium TypeScript, Visual AI Testing]
excerpt: >-
  Ensure your UI is flawless across every screen size by iterating through device viewports and capturing targeted mobile, tablet, and desktop visual baselines.
readTime: 4 min read
---

# Breakpoints and CSS: Automating Responsive Visual Tests

Over 60% of modern web traffic comes from mobile devices. Because of this, developers use **CSS Media Queries** to build "Responsive Web Design"—meaning the layout of the website changes dynamically based on the width of the user's browser. 

For example, a navigation bar that looks perfect on a 1920x1080 Desktop monitor will collapse into a "Hamburger Menu" on a 375x812 iPhone screen.

If you only execute your visual tests at Desktop dimensions, a catastrophic CSS bug that destroys the Mobile experience will slip right through your CI/CD pipeline!

In this tutorial, we will learn how to iterate through an array of device breakpoints and capture unique visual baselines for each resolution.

---

## 1. Defining the Viewports

Instead of writing three separate tests for Desktop, Tablet, and Mobile, we will define an array of viewports and use Jest's parameterized testing feature (`test.each`) to iterate over them automatically.

Create `tests/responsive_testing.test.ts`:

```typescript
import { Builder, WebDriver } from "selenium-webdriver";
import fs from "fs";
import path from "path";
// Define the critical CSS breakpoint dimensions for our application
const VIEWPORTS = [
  { name: "desktop", width: 1920, height: 1080 },
  { name: "tablet", width: 768, height: 1024 },
  { name: "mobile", width: 375, height: 812 }
];
describe("Phase 8 - Visual Testing: Responsive Testing", () => {
  let driver: WebDriver;
  const TARGET_URL = "https://practice.mycodeyatra.com/";
  beforeAll(async () => {
    driver = await new Builder().forBrowser("chrome").build();
  });
  afterAll(async () => {
    if (driver) {
      await driver.quit();
    }
  });
  // We use `test.each` to iterate through the array of viewports and run a separate test for each!
  test.each(VIEWPORTS)("Should verify the UI layout at $name ($width x $height)", async ({ name, width, height }) => {
    console.log(`[Visual] Testing Responsive Breakpoint: ${name}`);
    // 1. Resize the browser window to match the exact device breakpoint
    await driver.manage().window().setRect({ width, height });
    // 2. Navigate to the page. 
    // The browser will evaluate the viewport size and apply the correct Mobile/Tablet CSS!
    await driver.get(TARGET_URL);
    // 3. Take the screenshot for this specific resolution
    const screenshotBase64 = await driver.takeScreenshot();
    // 4. Save the baseline with the resolution injected into the filename!
    const screenshotsDir = path.resolve(__dirname, "__image_snapshots__");
    if (!fs.existsSync(screenshotsDir)) {
      fs.mkdirSync(screenshotsDir, { recursive: true });
    }
    // We create unique files: responsive_home_desktop.png, responsive_home_mobile.png, etc.
    const baselinePath = path.join(screenshotsDir, `responsive_home_${name}.png`);
    fs.writeFileSync(baselinePath, screenshotBase64, "base64");
    console.log(`[Visual] Captured responsive layout for ${name} at: ${baselinePath}`);
    expect(fs.existsSync(baselinePath)).toBe(true);
  });
});
```

---

## 2. Test Execution Output

Run the responsive visual suite:

```bash
> ENV_NAME=qa jest tests/responsive_testing.test.ts
```

Output:

```text
 PASS  tests/responsive_testing.test.ts
  Phase 8 - Visual Testing: Responsive Testing
    √ Should verify the UI layout at desktop (1920 x 1080) (2250 ms)
    √ Should verify the UI layout at tablet (768 x 1024) (1850 ms)
    √ Should verify the UI layout at mobile (375 x 812) (1710 ms)
  console.log
    [Visual] Testing Responsive Breakpoint: desktop
    [Visual] Captured responsive layout for desktop at: ...\responsive_home_desktop.png
    [Visual] Testing Responsive Breakpoint: tablet
    [Visual] Captured responsive layout for tablet at: ...\responsive_home_tablet.png
    [Visual] Testing Responsive Breakpoint: mobile
    [Visual] Captured responsive layout for mobile at: ...\responsive_home_mobile.png
```

If you look in your snapshots directory, you now have a matrix of visual baselines validating every single screen size your customers might use!

## Conclusion

Automated Responsive Testing guarantees that UI engineers cannot accidentally break the tablet experience while fixing a bug on the desktop experience. 

However, resizing a desktop browser window is not a perfect simulation of a real mobile device. In our next tutorial, we will explore true **Mobile Web Emulation**, configuring Selenium to mock mobile user-agents, touch events, and device pixel ratios!
