---
title: Capturing Screenshots & Visual Evidence in Selenium TypeScript
date: 07-Nov-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, screenshots, nodejs, visual-testing]
category: Selenium TypeScript
categories: [Selenium TypeScript, UI Automation, Architecture]
excerpt: >-
  When your automated test fails at 2:00 AM, you need visual evidence. Learn how to capture Base64 screenshots in Selenium TypeScript and write them to your local filesystem using Node.js!
readTime: 6 min read
---

# Capturing Screenshots & Visual Evidence in Selenium TypeScript

When an automated test runs on a headless server at 2:00 AM and fails, the terminal logs are rarely enough to debug the issue. You need visual evidence. Was the button missing? Did a popup block the screen? Did the CSS fail to load?

In this final article of our TypeScript Foundations series, we will learn how to capture raw Base64 image data from Google Chrome using Selenium WebDriver and save it to our local file system using Node.js!

---

## 1. How Selenium Captures Images

Under the hood, Selenium does not save `.png` or `.jpg` files directly to your hard drive. 

Instead, when you call `driver.takeScreenshot()`, the browser engine takes a snapshot of the current viewport, encodes the binary image data into a massive **Base64 String**, and sends that text back to Node.js. 

It is up to us, the automation engineers, to decode that Base64 string and write it to our local file system using Node's built-in `fs` (File System) module.

---

## 2. Writing the Screenshot Test

Let's write a script that navigates to **[practice.mycodeyatra.com](https://practice.mycodeyatra.com/)** and explicitly takes a screenshot. 

To keep our project organized, we will also write some Node.js logic to dynamically create a `screenshots` folder if one doesn't exist, and append a unique timestamp to the image filename so we don't overwrite previous runs!

Create `tests/screenshots.test.ts`:

```typescript
import { Builder, WebDriver } from "selenium-webdriver";
import * as fs from "fs";
import * as path from "path";
import "chromedriver";
describe("Visual Evidence - Screenshots", () => {
  let driver: WebDriver;
  beforeAll(async () => {
    // 1. Initialize Chrome
    driver = await new Builder().forBrowser("chrome").build();
    await driver.manage().window().maximize();
    // 2. Safely create a 'screenshots' directory using Node's fs module
    const screenshotsDir = path.join(__dirname, "screenshots");
    if (!fs.existsSync(screenshotsDir)) {
      fs.mkdirSync(screenshotsDir);
    }
  });
  afterAll(async () => {
    // Tear down Chrome
    if (driver) {
      await driver.quit();
    }
  });
  it("Should capture a base64 screenshot of the practice site", async () => {
    console.log("Navigating to https://practice.mycodeyatra.com/ ...");
    await driver.get("https://practice.mycodeyatra.com/");
    // 3. Command Selenium to capture the Base64 image
    const base64Image = await driver.takeScreenshot();
    // 4. Generate a unique filename using timestamps
    const timestamp = new Date().getTime();
    const filePath = path.join(__dirname, `screenshots/failure_${timestamp}.png`);
    // 5. Decode Base64 and Write to local disk
    fs.writeFileSync(filePath, base64Image, 'base64');
    console.log(`Screenshot successfully saved to: ${filePath}`);
    // 6. Assert that the file was physically created!
    expect(fs.existsSync(filePath)).toBe(true);
  });
});
```

---

## 3. Test Execution Output

When we execute `npm test tests/screenshots.test.ts`, Jest runs the test, connects to the browser, captures the Base64 payload, and writes the `.png` file directly to our hard drive:

```text
> mcyt-sel-typescript@1.0.0 test
> jest
 PASS  tests/screenshots.test.ts (7.112 s)
  Visual Evidence - Screenshots
    √ Should capture a base64 screenshot of the practice site (3502 ms)
  console.log
    Navigating to https://practice.mycodeyatra.com/ ...
  console.log
    Screenshot successfully saved to: D:\MyCodeYatra\AILearning2026\Repository\mcyt-sel-typescript\tests\screenshots\failure_1700000000000.png
Test Suites: 1 passed, 1 total
Tests:       1 passed, 1 total
Snapshots:   0 total
Time:        7.321 s
Ran all test suites.
```

## 4. What about Videos?
Unlike Cypress or Playwright, **Selenium does not have native video recording built-in.** 
If you want to record videos of your Selenium tests, you typically have to run them on a Cloud Grid (like BrowserStack) or use a third-party Docker container (like Selenoid) which records the VNC session via ffmpeg. 

## Conclusion

You now know how to capture, decode, and store visual evidence from your automated tests! In an Enterprise framework, you wouldn't take screenshots manually like this; instead, you would place this exact logic inside an `@afterEach` block, configuring Jest to *only* execute `takeScreenshot()` if the test failed.

### 🎉 The Selenium TypeScript Foundations Series is Complete!
Congratulations! You have successfully mastered the fundamentals of TypeScript, Node.js, Jest configurations, strict CSS Selectors, Chrome DevTools interception, and Visual Evidence gathering. You are now ready to graduate to the next phase: **Core UI Automation Mastery**.
