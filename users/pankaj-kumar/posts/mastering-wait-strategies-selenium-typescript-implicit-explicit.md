---
title: Mastering Wait Strategies in Selenium TypeScript (Implicit vs Explicit)
date: 08-Nov-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, wait-strategies, synchronization]
category: Selenium TypeScript
categories: [Selenium TypeScript, UI Automation, Core Automation]
excerpt: >-
  Stop using hardcoded sleeps! Learn how to dynamically synchronize your automation scripts using global Implicit Waits and precise Explicit Waits via the selenium-webdriver 'until' module.
readTime: 6 min read
---

# Mastering Wait Strategies in Selenium TypeScript (Implicit vs Explicit)

Welcome to Phase 2 of our Selenium TypeScript Masterclass: **Core UI Automation!**

Now that our framework is successfully running basic assertions and capturing visual evidence, it's time to tackle the number one cause of flaky automation tests: **Timing Issues.**

When a script tries to click a button before the JavaScript on the page has finished rendering it, Selenium throws a `NoSuchElementException`. In this article, we will use **[practice.mycodeyatra.com](https://practice.mycodeyatra.com/)** to learn how to solve this using Implicit and Explicit Waits.

---

## 1. The Evil `Thread.sleep()`

If you have ever written a test that fails randomly, you might have been tempted to add a hardcoded pause:

```typescript
// NEVER DO THIS!
await new Promise(resolve => setTimeout(resolve, 5000));
```
**Why is this terrible?**
If the element loads in 1 second, you are wasting 4 seconds of execution time. If you have 500 tests, that's almost an hour of wasted compute resources! We must dynamically wait *only as long as necessary*.

---

## 2. Implicit Waits: The Global Safety Net

An **Implicit Wait** tells Selenium WebDriver to poll the DOM for a certain amount of time when trying to find *any* element before throwing an exception.

```typescript
// Set implicit wait to 10 seconds
await driver.manage().setTimeouts({ implicit: 10000 });
```
Once configured, this applies globally for the entire lifespan of the `WebDriver` instance. If `driver.findElement()` cannot find the element immediately, it will automatically keep trying for up to 10 seconds. The moment it finds it, it proceeds instantly!

---

## 3. Explicit Waits: The Precision Scalpel

While Implicit Waits check if an element *exists in the HTML*, they do **not** check if the element is visible, clickable, or enabled. 

To wait for specific *states* (like waiting for a loading spinner to disappear before clicking a button), we use **Explicit Waits**. Selenium TypeScript provides the `until` module for this.

Let's write a script that implements both! Create `tests/wait_strategies.test.ts`:

```typescript
import { Builder, WebDriver, By, until } from "selenium-webdriver";
import "chromedriver";
describe("Core UI Automation - Wait Strategies", () => {
  let driver: WebDriver;
  beforeAll(async () => {
    driver = await new Builder().forBrowser("chrome").build();
    await driver.manage().window().maximize();
    // 1. Configure the Global Implicit Wait
    await driver.manage().setTimeouts({ implicit: 10000 });
  });
  afterAll(async () => {
    if (driver) {
      await driver.quit();
    }
  });
  it("Should handle Explicit Waits correctly using until modules", async () => {
    console.log("Navigating to https://practice.mycodeyatra.com/ ...");
    await driver.get("https://practice.mycodeyatra.com/");
    // 2. Define our CSS Locator
    const submitBtnLocator = By.css("button[type='submit']");
    console.log("Waiting for submit button to be visible...");
    // 3. Explicit Wait: Wait up to 15 seconds for visibility
    const submitBtn = await driver.wait(
      until.elementIsVisible(driver.findElement(submitBtnLocator)), 
      15000, 
      "Timed out after 15 seconds waiting for Submit Button to be visible"
    );
    // 4. Explicit Wait: Wait for element to be enabled before clicking
    await driver.wait(until.elementIsEnabled(submitBtn), 5000);
    const text = await submitBtn.getText();
    console.log(`Button successfully located and enabled: ${text}`);
    expect(text).toBeTruthy();
  });
});
```

---

## 4. Test Execution Output

```text
> mcyt-sel-typescript@1.0.0 test
> jest
 PASS  tests/wait_strategies.test.ts (6.411 s)
  Core UI Automation - Wait Strategies
    √ Should handle Explicit Waits correctly using until modules (3102 ms)
  console.log
    Navigating to https://practice.mycodeyatra.com/ ...
  console.log
    Waiting for submit button to be visible...
  console.log
    Button successfully located and enabled: Submit
Test Suites: 1 passed, 1 total
Tests:       1 passed, 1 total
Snapshots:   0 total
Time:        6.702 s
Ran all test suites.
```

## Conclusion

By combining a global Implicit Wait (to catch slow-loading DOM nodes) with highly precise Explicit Waits (to handle specific element states like visibility and clickability), you can completely eliminate flaky tests in your TypeScript framework.

In our next article, we will conquer one of the most tedious parts of UI Automation: **Handling Forms, Checkboxes, and Dropdowns!**
