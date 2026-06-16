---
title: Architecting UI Automation: The Page Object Model (POM) in TypeScript
date: 20-Nov-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, pom, page-object-model, architecture]
category: Selenium TypeScript
categories: [Selenium TypeScript, Architecture & Patterns]
excerpt: >-
  If your automation suite mixes WebDriver locators directly inside your test cases, you are building a house of cards. Learn how to architect the Page Object Model.
readTime: 6 min read
---

# Architecting UI Automation: The Page Object Model (POM) in TypeScript

If your automation suite mixes WebDriver locators directly inside your test cases, you are building a house of cards. 

What happens when the Development Team changes the ID of the Login button? If you have 50 tests that click that button, you will have to manually find and replace that string 50 times!

The **Page Object Model (POM)** is a structural design pattern that abstracts away the HTML of a webpage into a dedicated Class. If the UI changes, you only update the Class once, and all 50 tests will instantly inherit the fix.

In this tutorial, we will refactor our Forms testing logic into an elegant TypeScript Page Object.

---

## 1. Creating the Page Object Class

A Page Object consists of two distinct parts:
1. **Locators (Private variables):** The CSS or XPath strings used to find elements.
2. **Actions (Public methods):** The functions that tests can call to interact with the page.

Create a new file `tests/pages/FormsPage.ts`:

```typescript
import { WebDriver, By, until } from "selenium-webdriver";
export class FormsPage {
  private driver: WebDriver;
  // 1. Locators (Encapsulated)
  private readonly url = "https://practice.mycodeyatra.com/#/form-practice";
  private readonly firstNameInput = By.css("[data-testid='first-name']");
  private readonly lastNameInput = By.css("[data-testid='last-name']");
  private readonly emailInput = By.css("[data-testid='email']");
  private readonly mobileInput = By.css("[data-testid='mobile']");
  private readonly submitBtn = By.css("[data-testid='submit-btn']");
  private readonly resultMsg = By.css("[data-testid='result-message']");
  constructor(driver: WebDriver) {
    this.driver = driver;
  }
  // 2. Actions (Exposed to Tests)
  async navigate() {
    await this.driver.get(this.url);
  }
  async enterFirstName(name: string) {
    const el = await this.driver.wait(until.elementLocated(this.firstNameInput), 5000);
    await el.clear();
    await el.sendKeys(name);
  }
  async enterLastName(name: string) {
    const el = await this.driver.findElement(this.lastNameInput);
    await el.clear();
    await el.sendKeys(name);
  }
  async enterEmail(email: string) {
    const el = await this.driver.findElement(this.emailInput);
    await el.clear();
    await el.sendKeys(email);
  }
  async enterMobile(mobile: string) {
    const el = await this.driver.findElement(this.mobileInput);
    await el.clear();
    await el.sendKeys(mobile);
  }
  async submitForm() {
    const el = await this.driver.findElement(this.submitBtn);
    await el.click();
  }
  async getResultMessage(): Promise<string> {
    const el = await this.driver.wait(until.elementIsVisible(await this.driver.findElement(this.resultMsg)), 5000);
    return await el.getText();
  }
}
```

---

## 2. Writing the POM Test

Notice how our actual test script no longer contains a single CSS selector! 
The test reads like plain English, making it incredibly easy for Product Managers or junior QA engineers to review.

Create `tests/pom.test.ts`:

```typescript
import { WebDriver } from "selenium-webdriver";
import { DriverFactory } from "./utils/driverFactory";
import { FormsPage } from "./pages/FormsPage";
describe("Architecture Phase - Page Object Model", () => {
  let driver: WebDriver;
  let formsPage: FormsPage;
  beforeAll(async () => {
    driver = await DriverFactory.getDriver();
    // Initialize the Page Object
    formsPage = new FormsPage(driver);
  });
  afterAll(async () => {
    if (driver) {
      await driver.quit();
    }
  });
  it("Should successfully submit the form using Page Object Methods", async () => {
    console.log("Navigating to Forms Page...");
    await formsPage.navigate();
    console.log("Filling out user details...");
    await formsPage.enterFirstName("John");
    await formsPage.enterLastName("Doe");
    await formsPage.enterEmail("john.doe@example.com");
    await formsPage.enterMobile("1234567890");
    console.log("Submitting form...");
    await formsPage.submitForm();
    const msg = await formsPage.getResultMessage();
    expect(msg).toContain("Form submitted successfully");
    console.log(`Success! Result: ${msg}`);
  });
});
```

---

## 3. Test Execution Output

```text
> mcyt-sel-typescript@1.0.0 test
> jest tests/pom.test.ts
 PASS  tests/pom.test.ts (6.425 s)
  Architecture Phase - Page Object Model
    √ Should successfully submit the form using Page Object Methods (2104 ms)
  console.log
    Navigating to Forms Page...
  console.log
    Filling out user details...
  console.log
    Submitting form...
  console.log
    Success! Result: Form submitted successfully!
Test Suites: 1 passed, 1 total
Tests:       1 passed, 1 total
Snapshots:   0 total
Time:        6.650 s
Ran all test suites.
```

## Conclusion

The Page Object Model (POM) is not just a "nice-to-have"—it is an absolute necessity for any scalable test automation framework. By separating the *how* (locating elements) from the *what* (the business logic of the test), you drastically reduce maintenance overhead.

In our next tutorial, we will take our DriverFactory to the next level by implementing the **WebDriver Manager Pattern** for seamless environment configurations!
