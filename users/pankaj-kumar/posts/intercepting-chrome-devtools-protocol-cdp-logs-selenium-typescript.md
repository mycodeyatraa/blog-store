---
title: Intercepting Chrome DevTools & Logs in Selenium TypeScript
date: 06-Nov-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, cdp, chrome-devtools, logs]
category: Selenium TypeScript
categories: [Selenium TypeScript, UI Automation, Architecture]
excerpt: >-
  Stop relying only on UI validations. Learn how to tap directly into the Chrome DevTools Protocol (CDP) to capture hidden JavaScript errors and intercept network requests in Selenium 4.
readTime: 6 min read
---

# Intercepting Chrome DevTools & Logs in Selenium TypeScript

Have you ever written an automated test that passes perfectly, but the actual application is silently throwing JavaScript errors in the background? 

Standard UI automation only checks if elements exist on the screen. It does not check if the frontend React or Angular application is secretly failing. To build true Enterprise-grade automation, we need to look *behind* the UI and tap directly into the **Chrome DevTools Protocol (CDP)**.

In this tutorial, we will use Selenium TypeScript to capture browser console logs and network data directly from Chrome's backend engine while running tests on **[practice.mycodeyatra.com](https://practice.mycodeyatra.com/)**.

---

## 1. What is the Chrome DevTools Protocol?

When you press `F12` in Google Chrome, you open the DevTools panel (Elements, Console, Network, Performance). The Chrome DevTools Protocol (CDP) is the API that allows tools like Selenium (in version 4+) and Playwright to communicate directly with this backend.

By configuring Selenium to listen to CDP, we can:
- **Capture Console Errors:** Automatically fail a test if the frontend throws a `SEVERE` JavaScript exception.
- **Intercept Network Requests:** View API calls being made by the browser.
- **Mock Geolocation:** Trick the browser into thinking it's in a different country.

---

## 2. Configuring Logging Preferences in TypeScript

To access browser logs, we must explicitly enable them in our `chrome.Options` before the WebDriver is initialized.

```typescript
import { Builder, logging } from "selenium-webdriver";
import * as chrome from "selenium-webdriver/chrome";
// 1. Create a Logging Preferences object
const prefs = new logging.Preferences();
prefs.setLevel(logging.Type.BROWSER, logging.Level.ALL);
// 2. Attach it to Chrome Options
const options = new chrome.Options();
options.setLoggingPrefs(prefs);
// 3. Build the Driver
const driver = await new Builder()
  .forBrowser("chrome")
  .setChromeOptions(options)
  .build();
```

---

## 3. Writing the CDP Logging Test

Let's write a script that navigates to the MyCodeYatra Practice Site, injects some custom JavaScript to simulate a frontend crash, and then reads the logs to assert that the error was caught!

Create a new file: `tests/cdp_logs.test.ts`.

```typescript
import { Builder, WebDriver, logging } from "selenium-webdriver";
import * as chrome from "selenium-webdriver/chrome";
import "chromedriver";
describe("Chrome DevTools Protocol - Console Logs", () => {
  let driver: WebDriver;
  beforeAll(async () => {
    const prefs = new logging.Preferences();
    prefs.setLevel(logging.Type.BROWSER, logging.Level.ALL);
    const options = new chrome.Options();
    options.setLoggingPrefs(prefs);
    driver = await new Builder()
      .forBrowser("chrome")
      .setChromeOptions(options)
      .build();
    await driver.manage().window().maximize();
  });
  afterAll(async () => {
    if (driver) {
      await driver.quit();
    }
  });
  it("Should capture browser console logs on practice site", async () => {
    console.log("Navigating to https://practice.mycodeyatra.com/ ...");
    await driver.get("https://practice.mycodeyatra.com/");
    // Simulate frontend application activity
    await driver.executeScript("console.error('Simulated React Rendering Error!');");
    await driver.executeScript("console.log('User logged in successfully.');");
    // Fetch logs from the Chrome backend
    const logs = await driver.manage().logs().get(logging.Type.BROWSER);
    console.log(`Captured ${logs.length} log entries from Chrome.`);
    let hasError = false;
    for (const entry of logs) {
      console.log(`[${entry.level.name}] ${entry.message}`);
      if (entry.level.name === 'SEVERE') {
        hasError = true;
      }
    }
    // Assert that we successfully detected the SEVERE console error
    expect(hasError).toBe(true);
  });
});
```

---

## 4. Test Execution Output

When we execute `npm test tests/cdp_logs.test.ts`, Jest runs the test, connects to CDP, and successfully extracts the hidden console messages:

```text
> mcyt-sel-typescript@1.0.0 test
> jest
 PASS  tests/cdp_logs.test.ts (6.331 s)
  Chrome DevTools Protocol - Console Logs
    √ Should capture browser console logs on practice site (2801 ms)
  console.log
    Navigating to https://practice.mycodeyatra.com/ ...
  console.log
    Captured 2 log entries from Chrome.
  console.log
    [SEVERE] https://practice.mycodeyatra.com/ 1:8 "Simulated React Rendering Error!"
  console.log
    [INFO] https://practice.mycodeyatra.com/ 1:8 "User logged in successfully."
Test Suites: 1 passed, 1 total
Tests:       1 passed, 1 total
Snapshots:   0 total
Time:        6.541 s
Ran all test suites.
```

## Conclusion

By leveraging Selenium 4's integration with the Chrome DevTools Protocol, we have elevated our testing framework from simple "UI clicking" to deep "Application State Monitoring". You can now build custom Jest assertions that automatically fail tests if any `SEVERE` HTTP 500 errors or JavaScript exceptions appear in the console.

In our next and final article for the TypeScript Foundations series, we will learn how to capture **Screenshots and Videos** of our test executions!
