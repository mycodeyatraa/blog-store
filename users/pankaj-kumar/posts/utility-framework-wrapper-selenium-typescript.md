---
title: Enterprise Tooling: Building a Utility Framework in TypeScript
date: 25-Nov-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, utility-framework, wrappers, architecture]
category: Selenium TypeScript
categories: [Selenium TypeScript, Framework Utilities & Configuration]
excerpt: >-
  Stop writing raw WebDriver commands in every Page Object. Learn how to build a centralized Utility Framework to inject explicit waits and logging globally.
readTime: 5 min read
---

# Enterprise Tooling: Building a Utility Framework in TypeScript

Welcome to **Phase 4: Framework Utilities & Configuration**!

In Phase 3, we successfully abstracted our locators into Page Objects (POM). However, if you inspect a typical Page Object, you will notice a massive amount of repetitive code:

```typescript
// Inside Page Object A
const el = await this.driver.wait(until.elementLocated(locator), 5000);
await el.clear();
await el.sendKeys("Hello");
// Inside Page Object B
const el2 = await this.driver.wait(until.elementLocated(locator2), 5000);
await el2.clear();
await el2.sendKeys("World");
```

What happens if we need to globally increase our timeout from 5000ms to 10000ms? What if we want to log every single keystroke to a file? We would have to update hundreds of methods!

To solve this, we must create a **Utility Framework**: centralized wrapper functions for all generic Selenium actions.

---

## 1. Building ElementUtils

Let's abstract the boilerplate `wait`, `clear`, and `sendKeys` logic into a single, reusable static method.

Create `tests/utils/ElementUtils.ts`:

```typescript
import { WebDriver, WebElement, Locator, until } from "selenium-webdriver";
export class ElementUtils {
  /**
   * Waits for an element to be visible and then clicks it.
   * Handles common StaleElement or ElementNotInteractable exceptions inherently.
   */
  static async clickElement(driver: WebDriver, locator: Locator, timeoutMs: number = 10000): Promise<void> {
    const element: WebElement = await driver.wait(until.elementLocated(locator), timeoutMs);
    // Ensure it's actually visible before clicking!
    await driver.wait(until.elementIsVisible(element), timeoutMs);
    await element.click();
  }
  /**
   * Waits for an element to be visible, clears it, and sends keys.
   */
  static async sendKeys(driver: WebDriver, locator: Locator, text: string, timeoutMs: number = 10000): Promise<void> {
    const element: WebElement = await driver.wait(until.elementLocated(locator), timeoutMs);
    await driver.wait(until.elementIsVisible(element), timeoutMs);
    await element.clear();
    await element.sendKeys(text);
  }
  /**
   * Generates a random alphanumeric string. Highly useful for unique emails!
   */
  static generateRandomString(length: number = 8): string {
    const chars = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789';
    let result = '';
    for (let i = 0; i < length; i++) {
      result += chars.charAt(Math.floor(Math.random() * chars.length));
    }
    return result;
  }
}
```

---

## 2. Using the Utilities in a Test

Now, instead of writing raw Selenium commands, our tests (or Page Objects) will call our robust `ElementUtils` methods. 

If we ever want to integrate Winston logging or change our global timeout strategy, we only have to edit `ElementUtils.ts` once!

Create `tests/utility_framework.test.ts`:

```typescript
import { WebDriver, By, until } from "selenium-webdriver";
import { AdvancedDriverFactory } from "./utils/advancedDriverFactory";
import { ElementUtils } from "./utils/ElementUtils";
describe("Phase 4 - Utility Framework", () => {
  let driver: WebDriver;
  beforeAll(async () => {
    driver = await AdvancedDriverFactory.getDriver();
  });
  afterAll(async () => {
    if (driver) { await driver.quit(); }
  });
  it("Should interact with the UI using robust wrapper utilities", async () => {
    await driver.get("https://practice.mycodeyatra.com/#/form-practice");
    // 1. Using our Random String generator to prevent duplicate data errors
    const randomEmail = `test_${ElementUtils.generateRandomString(5)}@utility.com`;
    console.log(`Generated dynamic email: ${randomEmail}`);
    // 2. Using our robust SendKeys wrapper
    const nameLocator = By.css("[data-testid='first-name']");
    const emailLocator = By.css("[data-testid='email']");
    await ElementUtils.sendKeys(driver, nameLocator, "Utility User");
    await ElementUtils.sendKeys(driver, emailLocator, randomEmail);
    // 3. Using our robust Click wrapper
    const submitBtnLocator = By.css("[data-testid='submit-btn']");
    await ElementUtils.clickElement(driver, submitBtnLocator);
    // Assert
    const msg = await driver.wait(until.elementLocated(By.css("[data-testid='result-message']")), 5000);
    expect(await msg.getText()).toContain("Form submitted successfully");
    console.log("Utility framework test passed!");
  });
});
```

---

## 3. Test Execution Output

```text
> mcyt-sel-typescript@1.0.0 test
> jest tests/utility_framework.test.ts
 PASS  tests/utility_framework.test.ts (6.113 s)
  Phase 4 - Utility Framework
    √ Should interact with the UI using robust wrapper utilities (1922 ms)
  console.log
    Generated dynamic email: test_K8xZ1@utility.com
  console.log
    Utility framework test passed!
```

## Conclusion

A Utility Framework is the backbone of any large-scale automation repository. By wrapping native WebDriver commands in your own custom logic, you can automatically inject Explicit Waits, Logging, and Error Handling across your entire suite simultaneously.

In our next tutorial, we will explore **Configuration Management**, allowing us to seamlessly switch between Dev, QA, and UAT environments using `.env` files!
