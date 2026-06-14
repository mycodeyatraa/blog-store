---
title: Bypassing the Shadow DOM in Selenium TypeScript
date: 14-Nov-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, shadow-dom, web-components]
category: Selenium TypeScript
categories: [Selenium TypeScript, UI Automation, Core Automation]
excerpt: >-
  If an element lives inside a Shadow DOM, standard Selenium locators will fail. Learn how to pierce the Shadow Root and interact with encapsulated Web Components.
readTime: 6 min read
---

# Bypassing the Shadow DOM in Selenium TypeScript

As modern web applications migrate toward Web Components (like Salesforce Lightning), Automation Engineers are encountering a brand new nightmare: **The Shadow DOM**.

The Shadow DOM is a feature of HTML that encapsulates styles and markup. If an element lives inside a Shadow DOM, standard Selenium locators like `By.css` or `By.xpath` will fail to find it, returning a `NoSuchElementError`.

In this tutorial, we will learn how to pierce the Shadow DOM and interact with encapsulated elements using TypeScript on **[practice.mycodeyatra.com/#/shadow-dom](https://practice.mycodeyatra.com/#/shadow-dom)**.

---

## 1. What is a Shadow Host?

To access a Shadow DOM, you must first find its **Shadow Host**. 
The Shadow Host is the standard HTML element in the main document that the Shadow DOM is attached to.

Because the Shadow Host is part of the regular DOM, you can locate it normally!

```typescript
// Locate the Shadow Host element (Normal DOM)
const shadowHost = await driver.findElement(By.css("[data-testid='shadow-host']"));
```

---

## 2. Piercing the Shadow Root

In Selenium 4, a native method was introduced to retrieve the Shadow Root from a Host element: `.getShadowRoot()`.

Once you have the `shadowRoot` object, you can treat it exactly like the `driver` object! You can call `.findElement()` on it to search exclusively within the encapsulated DOM.

*Note: XPath does not work inside a Shadow Root! You must use CSS Selectors.*

```typescript
// Get the Shadow Root
const shadowRoot = await shadowHost.getShadowRoot();
// Search inside the Shadow DOM using CSS
const shadowInput = await shadowRoot.findElement(By.css("#shadow-input"));
```

---

## 3. Writing the Shadow DOM Test

Let's write a complete Jest test. We will navigate to the Sandbox, pierce the Shadow Root, fill out a text input hidden inside it, click a hidden button, and assert the resulting dynamic text.

Create `tests/shadow_dom.test.ts`:

```typescript
import { Builder, WebDriver, By, until } from "selenium-webdriver";
import "chromedriver";
describe("Core UI Automation - Shadow DOM", () => {
  let driver: WebDriver;
  beforeAll(async () => {
    driver = await new Builder().forBrowser("chrome").build();
    await driver.manage().window().maximize();
    await driver.manage().setTimeouts({ implicit: 5000 });
  });
  afterAll(async () => {
    if (driver) {
      await driver.quit();
    }
  });
  it("Should pierce the Shadow DOM and interact with encapsulated elements", async () => {
    console.log("Navigating to Shadow DOM Page...");
    await driver.get("https://practice.mycodeyatra.com/#/shadow-dom");
    // Wait for the Shadow Host to be located
    const shadowHost = await driver.wait(
      until.elementLocated(By.css("[data-testid='shadow-host']")), 
      5000
    );
    // Get the Shadow Root from the Host
    const shadowRoot = await shadowHost.getShadowRoot();
    console.log("Successfully pierced the Shadow Root!");
    // Interact with the input field INSIDE the Shadow DOM
    const shadowInput = await shadowRoot.findElement(By.css("#shadow-input"));
    await shadowInput.clear();
    await shadowInput.sendKeys("Hello from TypeScript Automation!");
    console.log("Filled text inside the Shadow DOM input");
    // Click the button INSIDE the Shadow DOM
    const shadowBtn = await shadowRoot.findElement(By.css("#shadow-btn"));
    await shadowBtn.click();
    console.log("Clicked the button inside the Shadow DOM");
    // Verify the resulting message INSIDE the Shadow DOM
    const shadowMsg = await shadowRoot.findElement(By.css("#shadow-msg"));
    const msgText = await shadowMsg.getText();
    expect(msgText).toEqual("You typed: Hello from TypeScript Automation!");
    console.log(`Verified output message: ${msgText}`);
  });
});
```

---

## 4. Test Execution Output

```text
> mcyt-sel-typescript@1.0.0 test
> jest
 PASS  tests/shadow_dom.test.ts (6.113 s)
  Core UI Automation - Shadow DOM
    √ Should pierce the Shadow DOM and interact with encapsulated elements (1852 ms)
  console.log
    Navigating to Shadow DOM Page...
  console.log
    Successfully pierced the Shadow Root!
  console.log
    Filled text inside the Shadow DOM input
  console.log
    Clicked the button inside the Shadow DOM
  console.log
    Verified output message: You typed: Hello from TypeScript Automation!
Test Suites: 1 passed, 1 total
Tests:       1 passed, 1 total
Snapshots:   0 total
Time:        6.313 s
Ran all test suites.
```

## Conclusion

The Shadow DOM is intimidating at first, but with Selenium 4's `getShadowRoot()` method, it becomes trivial. 
Just remember:
1. Find the normal Shadow Host element.
2. Call `getShadowRoot()`.
3. Use `shadowRoot.findElement(By.css("..."))` (Never use XPath!).

In our next tutorial, we will explore an incredibly advanced feature introduced in Selenium 4: **Intercepting the Chrome DevTools Network (CDP)!**
