---
title: Mastering Web Locators in Selenium TypeScript (Strict CSS Selectors)
date: 04-Nov-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, locators, css-selectors, shadow-dom]
category: Selenium TypeScript
categories: [Selenium TypeScript, UI Automation, Architecture]
excerpt: >-
  Ditch XPath forever! Learn why modern UI automation frameworks strictly use CSS Selectors to pierce the Shadow DOM, increase execution speed, and improve code readability.
readTime: 6 min read
---

# Mastering Web Locators in Selenium TypeScript (Strict CSS Selectors)

If you cannot find an element on the screen, you cannot automate it. Locators are the absolute foundation of UI Automation. 

In this tutorial, we will use our official testing sandbox—**[practice.mycodeyatra.com](https://practice.mycodeyatra.com/)**—to learn how to locate web elements using Selenium TypeScript.

---

## 1. Why We Only Use CSS Selectors (No XPath)

If you have written Java or Python automation in the past, you probably used a lot of `XPath` (`//div[@class='xyz']`). 

**In our Enterprise TypeScript Frameworks, we strictly forbid the use of XPath. We use CSS Selectors exclusively.** Why?
1. **Speed:** Browsers parse CSS Selectors natively via their CSS engines. They are measurably faster than XPath engines.
2. **Shadow DOM Compatibility:** Modern web components (like Salesforce LWC or React custom elements) hide their internals inside a `Shadow DOM`. **XPath cannot pierce the Shadow DOM.** Only CSS Selectors can traverse into shadow boundaries using `shadowRoot`.
3. **Readability:** CSS syntax is much cleaner and shorter than XPath.

---

## 2. Basic CSS Selectors

Selenium provides the `By` class to define locators. Because we are exclusively using CSS, we will only use `By.css()`.

- **By ID (`#`):** Finds an element with a specific ID. `By.css("#name")`
- **By Class (`.`):** Finds an element with a specific class. `By.css(".email-field")`
- **By Attribute (`[]`):** Finds an element by a specific attribute. `By.css("input[placeholder='Enter Name']")`

---

## 3. Writing the Locators Test

Let's write a real script that targets elements on the MyCodeYatra practice site.

Create a new file in your project: `tests/locators.test.ts`.

```typescript
import { Builder, WebDriver, By } from "selenium-webdriver";
import "chromedriver";
describe("Web Locators using CSS Selectors", () => {
  let driver: WebDriver;
  beforeAll(async () => {
    // Initialize Chrome
    driver = await new Builder().forBrowser("chrome").build();
    await driver.manage().window().maximize();
  });
  afterAll(async () => {
    // Tear down Chrome
    if (driver) {
      await driver.quit();
    }
  });
  it("Should locate elements using advanced CSS Selectors", async () => {
    console.log("Navigating to https://practice.mycodeyatra.com/ ...");
    await driver.get("https://practice.mycodeyatra.com/");
    // 1. Using ID via CSS (#name)
    const nameInput = await driver.findElement(By.css("#name"));
    await nameInput.sendKeys("John Doe");
    console.log("Successfully typed into #name");
    // 2. Using Class via CSS (.email-field)
    const emailInput = await driver.findElement(By.css(".email-field"));
    await emailInput.sendKeys("john@example.com");
    console.log("Successfully typed into .email-field");
    // 3. Using Attributes via CSS ([type='submit'])
    const submitBtn = await driver.findElement(By.css("button[type='submit']"));
    const btnText = await submitBtn.getText();
    console.log(`Submit button text found: ${btnText}`);
    // Jest Assertion
    expect(btnText).toBeTruthy();
  });
});
```

---

## 4. Test Execution Output

When we run this test via `npm test`, Jest compiles our TypeScript on the fly, launches Chrome, performs the operations using our strict CSS locators, and provides the following terminal output:

```text
> mcyt-sel-typescript@1.0.0 test
> jest
 PASS  tests/locators.test.ts (6.241 s)
  Web Locators using CSS Selectors
    √ Should locate elements using advanced CSS Selectors (3805 ms)
  console.log
    Navigating to https://practice.mycodeyatra.com/ ...
  console.log
    Successfully typed into #name
  console.log
    Successfully typed into .email-field
  console.log
    Submit button text found: Submit
Test Suites: 1 passed, 1 total
Tests:       1 passed, 1 total
Snapshots:   0 total
Time:        6.512 s
Ran all test suites.
```

## Conclusion

By strictly adhering to CSS Selectors (`By.css`), our TypeScript automation framework will remain fast, readable, and perfectly compatible with modern web components like the Shadow DOM.

In our next article, we will take a deep dive into **Browser Interactions**—learning how to handle dropdowns, checkboxes, alerts, and complex mouse hover actions!
