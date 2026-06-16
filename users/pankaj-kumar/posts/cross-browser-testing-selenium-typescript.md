---
title: True Automation Matrix: Cross-Browser Testing in TypeScript
date: 18-Nov-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, cross-browser, firefox, edge, matrix]
category: Selenium TypeScript
categories: [Selenium TypeScript, UI Automation, Core Automation]
excerpt: >-
  The true power of Selenium WebDriver is executing the exact same code across multiple browser engines. Learn how to implement a DriverFactory for cross-browser testing.
readTime: 5 min read
---

# True Automation Matrix: Cross-Browser Testing in TypeScript

You've built an incredible UI Automation suite that runs flawlessly in Chrome. But what happens when a user opens your web app in Firefox or Microsoft Edge? Do the CSS dropdowns still work? Do the `click()` events still fire?

The true power of Selenium WebDriver is that it is a W3C standard. You can execute the exact same TypeScript code across multiple browser engines!

In this tutorial, we will refactor our framework to support **Dynamic Cross-Browser Testing** using Environment Variables.

---

## 1. The Problem with Hardcoding

In our previous tests, we initialized our driver like this:

```typescript
driver = await new Builder().forBrowser("chrome").build();
```

If we want to run our suite on Firefox, we would have to manually edit all 10 of our test files and change `"chrome"` to `"firefox"`. This is highly inefficient.

Instead, we need to extract the browser instantiation logic into a central, reusable utility class.

---

## 2. Creating the Driver Factory

Let's create a new file at `tests/utils/driverFactory.ts`. This class will read a Node.js Environment Variable (`process.env.BROWSER`) to dynamically determine which browser to launch.

```typescript
import { Builder, WebDriver } from "selenium-webdriver";
import "chromedriver";
import "geckodriver"; // Required for Firefox
export class DriverFactory {
  static async getDriver(): Promise<WebDriver> {
    // Read the browser from the environment variable, default to 'chrome'
    const browserName = (process.env.BROWSER || "chrome").toLowerCase();
    let driver = await new Builder().forBrowser(browserName).build();
    await driver.manage().window().maximize();
    await driver.manage().setTimeouts({ implicit: 5000 });
    return driver;
  }
}
```

*Note: You must install the browser drivers (e.g., `npm install geckodriver`) for this to work!*

---

## 3. Implementing the Factory in our Tests

Now, we can update our Jest tests to call `DriverFactory.getDriver()` instead of hardcoding the Builder.

Create `tests/cross_browser.test.ts`:

```typescript
import { WebDriver, By, until } from "selenium-webdriver";
import { DriverFactory } from "./utils/driverFactory";
describe("Core UI Automation - Cross-Browser Testing", () => {
  let driver: WebDriver;
  beforeAll(async () => {
    // Instantiate driver dynamically
    driver = await DriverFactory.getDriver();
  });
  afterAll(async () => {
    if (driver) {
      await driver.quit();
    }
  });
  it("Should load the Practice Site in the specified browser", async () => {
    const currentBrowserName = (await driver.getCapabilities()).getBrowserName();
    console.log(`Executing test on: ${currentBrowserName.toUpperCase()}`);
    await driver.get("https://practice.mycodeyatra.com/");
    // Verify the homepage loaded successfully
    const header = await driver.wait(
      until.elementLocated(By.css("h1")), 
      5000
    );
    const headerText = await header.getText();
    expect(headerText).toContain("Practice Automation");
    console.log(`Successfully verified UI on ${currentBrowserName}!`);
  });
});
```

---

## 4. Test Execution via Command Line

By utilizing Environment Variables, we can now orchestrate our test runs directly from the terminal!

**Run on Chrome (Default):**

```bash
> jest tests/cross_browser.test.ts
```

**Run on Firefox:**

```bash
# Windows PowerShell
> $env:BROWSER="firefox"; jest tests/cross_browser.test.ts
# Mac/Linux Bash
> BROWSER=firefox jest tests/cross_browser.test.ts
```

**Run on Microsoft Edge:**

```bash
# Windows PowerShell
> $env:BROWSER="MicrosoftEdge"; jest tests/cross_browser.test.ts
```

## Conclusion

Congratulations! By implementing a `DriverFactory` and utilizing `process.env`, your TypeScript automation suite is now Enterprise-Ready. You can easily plug this framework into a Jenkins or GitHub Actions pipeline and define a matrix build to execute your suite simultaneously across Chrome, Firefox, and Edge!

This officially completes **Phase 2: Core UI Automation** of our Selenium TypeScript Mastery series!
