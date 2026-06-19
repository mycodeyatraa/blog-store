---
title: The End of Maintenance: AI Locator Healing
date: 16-Feb-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, ai, self-healing, locators, machine-learning, maintenance]
category: Selenium TypeScript
categories: [Selenium TypeScript, AI in Testing]
excerpt: >-
  Solve the oldest problem in automation. Learn how to wrap WebDriver in a custom TypeScript HealNet class that uses Machine Learning to dynamically fix broken CSS selectors at runtime.
readTime: 4 min read
---

# The End of Maintenance: AI Locator Healing

Since the invention of Selenium in 2004, the software testing industry has been plagued by a single, inescapable problem: **Flaky Locators**.

You write a perfect test. You use a robust CSS selector like `#submit-login-btn`. The test passes 100 times in a row. 

Then, a Frontend Developer refactors the UI and changes the ID to `#btn-login-submit`. The test fails. The CI/CD pipeline stops. The release is blocked. You are paged at 2:00 AM to fix a broken string.

This is the definition of **Maintenance Overhead**. 

With the advent of AI, we can finally solve this problem forever using **Self-Healing Automation**.

---

## 1. What is a Self-Healing Locator?

A traditional WebDriver script throws a `NoSuchElementException` when it cannot find a locator and instantly crashes the test.

An AI-driven **Self-Healing** script catches that exception. Before crashing, it analyzes the entire Document Object Model (DOM) of the current webpage. It uses a Machine Learning algorithm to look for elements that *look similar* to the element you were trying to find.

If it finds a match (e.g., `#btn-login-submit` has the exact same inner text and CSS classes as the old `#submit-login-btn`), it silently updates the locator, clicks the button, and allows the test to pass!

---

## 2. Implementing a Healing Wrapper in TypeScript

To build a self-healing engine in TypeScript, we must override the default `driver.findElement()` behavior.

We create a custom `HealNet` class that wraps our interactions:

```typescript
import { WebDriver, By, WebElement } from 'selenium-webdriver';
import axios from 'axios';
export class HealNet {
  private driver: WebDriver;
  constructor(driver: WebDriver) {
    this.driver = driver;
  }
  async clickElement(locator: By, elementName: string): Promise<void> {
    try {
      // 1. Try the standard Selenium click
      const element = await this.driver.findElement(locator);
      await element.click();
    } catch (error) {
      if (error.name === 'NoSuchElementError') {
        console.warn(`[HEAL-NET] Locator failed for ${elementName}. Initiating self-healing...`);
        // 2. Extract the entire DOM
        const pageSource = await this.driver.getPageSource();
        // 3. Send the DOM and the failed intent to an LLM API
        const newLocator = await this.askAIForNewLocator(pageSource, elementName);
        if (newLocator) {
          console.log(`[HEAL-NET] AI found new locator: ${newLocator}. Retrying...`);
          const healedElement = await this.driver.findElement(By.css(newLocator));
          await healedElement.click();
          // 4. Important: Log the heal so engineers can update the POM tomorrow!
          this.reportHealedLocator(elementName, locator.toString(), newLocator);
        } else {
          throw new Error(`[HEAL-NET] AI could not heal the locator for ${elementName}`);
        }
      } else {
        throw error;
      }
    }
  }
  // Helper method calling OpenAI/Gemini
  private async askAIForNewLocator(dom: string, elementIntent: string): Promise<string | null> {
      // (Implementation calls the LLM API with the DOM and asks for the CSS selector)
      return "#btn-login-submit"; // Mocked AI response
  }
}
```

---

## 3. The Page Object Integration

Now, you update your Page Object Models to use the `HealNet` class instead of raw WebDriver commands:

```typescript
export class LoginPage {
  private healer: HealNet;
  private submitBtn = By.css('#submit-login-btn');
  constructor(driver: WebDriver) {
    this.healer = new HealNet(driver);
  }
  async clickSubmit() {
    // Instead of: await this.driver.findElement(this.submitBtn).click();
    await this.healer.clickElement(this.submitBtn, "Login Submit Button");
  }
}
```

When the developer changes the ID, the test doesn't crash. It pauses for 2 seconds, prints `[HEAL-NET] AI found new locator...` to the console, and passes successfully!

---

## 4. The Danger of Silent Heals

While self-healing is magical, it introduces a dangerous architectural flaw: **Silent Degradation**.

If the AI heals a broken locator, your test passes. But the locator in your codebase (`#submit-login-btn`) is still wrong! The next time the test runs, the AI has to heal it *again*, slowing down execution.

This is why step 4 in the code above (`reportHealedLocator`) is so critical. You must integrate your healing engine with Jira or Slack. 

The test should pass, but it must send an asynchronous Slack message saying: *"Warning: The Login Submit Button locator broke. The AI healed it using `#btn-login-submit`. Please update the `LoginPage.ts` file."*

## Conclusion

By wrapping our WebDriver interactions in an AI-powered error handler, we have eliminated the maintenance nightmare of flaky locators. Our pipeline is now completely resilient to minor UI changes.

But integrating direct HTTP calls to OpenAI inside a `try/catch` block is a bit primitive. 

What if there was a standardized protocol for AIs to interface with the DOM? In our next tutorial, we will explore the future of tool orchestration: **MCP Integration**!
