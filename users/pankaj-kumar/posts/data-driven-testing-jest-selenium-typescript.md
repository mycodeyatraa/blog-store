---
title: Advanced Architecture: Data Driven Testing in Jest
date: 19-Nov-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, data-driven-testing, ddt, jest]
category: Selenium TypeScript
categories: [Selenium TypeScript, Architecture & Patterns]
excerpt: >-
  A major anti-pattern in automation is duplicating test logic. Learn how to feed JSON datasets directly into Jest to automatically generate test cases at runtime.
readTime: 5 min read
---

# Advanced Architecture: Data Driven Testing in Jest

Welcome to **Phase 3** of the Selenium TypeScript Mastery series! We are shifting our focus from "how to click buttons" to "how to build Enterprise-grade frameworks."

A major anti-pattern in test automation is duplicating code. If you need to test a Login Form with 10 different combinations of valid and invalid users, you should *not* write 10 different `it()` blocks!

Instead, you should separate your test logic from your test data. This is known as **Data Driven Testing (DDT)**. 

In this tutorial, we will learn how to feed JSON datasets directly into Jest to automatically generate test cases at runtime.

---

## 1. Creating the Test Data

First, we isolate our permutations into a dedicated data file. Create a file named `tests/data/users.json`.

```json
[
  {
    "scenario": "Valid Admin",
    "username": "alice_admin",
    "email": "alice@mycodeyatra.com",
    "expectedMessage": "User alice_admin created successfully."
  },
  {
    "scenario": "Valid QA",
    "username": "bob_tester",
    "email": "bob@mycodeyatra.com",
    "expectedMessage": "User bob_tester created successfully."
  },
  {
    "scenario": "Invalid Email Format",
    "username": "charlie",
    "email": "charlie_at_gmail.com",
    "expectedMessage": "Invalid email format."
  }
]
```

---

## 2. Parameterizing Jest using `test.each`

Jest natively supports Data Driven Testing via the `test.each()` syntax. 

You pass your imported JSON array into `test.each(myArray)`, and Jest will iterate through it, executing your test block once for every object in the array!

Create `tests/data_driven.test.ts`:

```typescript
import { WebDriver, By, until } from "selenium-webdriver";
import { DriverFactory } from "./utils/driverFactory";
// 1. Import the JSON Data
import * as testData from "./data/users.json";
describe("Architecture Phase - Data Driven Testing", () => {
  let driver: WebDriver;
  beforeAll(async () => {
    driver = await DriverFactory.getDriver();
  });
  afterAll(async () => {
    if (driver) {
      await driver.quit();
    }
  });
  // 2. Feed the data array into test.each()
  test.each(testData)("Should test scenario: $scenario", async (data) => {
    console.log(`Executing DDT Scenario: ${data.scenario}`);
    await driver.get("https://practice.mycodeyatra.com/#/form-practice");
    // 3. Use 'data.username' and 'data.email' dynamically!
    const nameInput = await driver.wait(
      until.elementLocated(By.css("[data-testid='first-name']")), 
      5000
    );
    await nameInput.clear();
    await nameInput.sendKeys(data.username);
    const emailInput = await driver.findElement(By.css("[data-testid='email']"));
    await emailInput.clear();
    await emailInput.sendKeys(data.email);
    console.log(`Expected outcome: ${data.expectedMessage}`);
    // Validate
    const actualEmail = await emailInput.getAttribute("value");
    expect(actualEmail).toEqual(data.email);
  });
});
```

*(Note: Ensure `"resolveJsonModule": true` is enabled in your `tsconfig.json`!)*

---

## 3. Test Execution Output

When you run this script, Jest automatically unpacks the JSON array and registers **three separate test cases** dynamically!

```text
> mcyt-sel-typescript@1.0.0 test
> jest tests/data_driven.test.ts
 PASS  tests/data_driven.test.ts (9.845 s)
  Architecture Phase - Data Driven Testing
    √ Should test scenario: Valid Admin (1560 ms)
    √ Should test scenario: Valid QA (1430 ms)
    √ Should test scenario: Invalid Email Format (1390 ms)
  console.log
    Executing DDT Scenario: Valid Admin
  console.log
    Expected outcome: User alice_admin created successfully.
  console.log
    Executing DDT Scenario: Valid QA
  console.log
    Expected outcome: User bob_tester created successfully.
  console.log
    Executing DDT Scenario: Invalid Email Format
  console.log
    Expected outcome: Invalid email format.
Test Suites: 1 passed, 1 total
Tests:       3 passed, 3 total
```

## Conclusion

Data Driven Testing drastically reduces code duplication and separates concerns. Product Owners or Manual QA engineers can now update `users.json` or `users.csv` to add hundreds of new edge-case tests without ever touching your TypeScript code!

In our next tutorial, we will tackle the holy grail of UI Automation design patterns: **The Page Object Model (POM)!**
