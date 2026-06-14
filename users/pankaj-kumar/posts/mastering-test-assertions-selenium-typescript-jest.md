---
title: Mastering Test Assertions in Selenium TypeScript using Jest
date: 05-Nov-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, jest, assertions, validation]
category: Selenium TypeScript
categories: [Selenium TypeScript, UI Automation, Architecture]
excerpt: >-
  A test script without assertions is useless. Learn how to use Jest's expect() matchers to rigorously validate web elements, check visibility states, and assert text values in Selenium TypeScript.
readTime: 6 min read
---

# Mastering Test Assertions in Selenium TypeScript using Jest

A test script without assertions is not an automated test—it is just a script that clicks buttons blindly. The true value of UI Automation lies in **Assertions**: verifying that the application is in the expected state after an action is performed.

In Java, you might use TestNG or JUnit assertions. In Python, you might use Pytest's `assert` keyword. In the TypeScript ecosystem, the industry standard is **Jest**. 

In this tutorial, we will learn how to use Jest's `expect()` library to validate web elements on our official practice sandbox: **[practice.mycodeyatra.com](https://practice.mycodeyatra.com/)**.

---

## 1. What is Jest's `expect()`?

Jest provides a massive library of "matchers" that allow you to validate different types of data.
When validating Web Elements in Selenium, we primarily care about three things:
1. **Visibility:** Is the element displayed on the screen? (`expect(boolean).toBe(true)`)
2. **Text Verification:** Does the element contain the correct text? (`expect(string).toEqual("Expected Text")`)
3. **State:** Is a button enabled or disabled? Is a checkbox checked?

---

## 2. Writing the Assertions Test

Let's write a comprehensive script that navigates to the MyCodeYatra practice form and verifies multiple conditions using strict CSS selectors.

Create a new file in your project: `tests/assertions.test.ts`.

```typescript
import { Builder, WebDriver, By } from "selenium-webdriver";
import "chromedriver";
describe("Test Assertions using Jest", () => {
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
  it("Should assert element visibility and text on MyCodeYatra Practice", async () => {
    console.log("Navigating to https://practice.mycodeyatra.com/ ...");
    await driver.get("https://practice.mycodeyatra.com/");
    // ----------------------------------------------------
    // Assertion 1: Verify Page Title Contains Specific Text
    // ----------------------------------------------------
    const title = await driver.getTitle();
    expect(title).toContain("MyCodeYatra");
    console.log("Assertion 1 Passed: Title is correct");
    // ----------------------------------------------------
    // Assertion 2: Verify Element is Displayed on Screen
    // ----------------------------------------------------
    const headerElement = await driver.findElement(By.css("h1"));
    const isDisplayed = await headerElement.isDisplayed();
    expect(isDisplayed).toBe(true);  // Strict boolean match
    console.log("Assertion 2 Passed: Header is visible");
    // ----------------------------------------------------
    // Assertion 3: Verify Exact Element Text
    // ----------------------------------------------------
    const headerText = await headerElement.getText();
    expect(headerText).toEqual("Automation Practice Form"); // Strict string match
    console.log(`Assertion 3 Passed: Header text is '${headerText}'`);
    // ----------------------------------------------------
    // Assertion 4: Verify Input Field is Enabled
    // ----------------------------------------------------
    const nameInput = await driver.findElement(By.css("#name"));
    const isEnabled = await nameInput.isEnabled();
    expect(isEnabled).toBe(true);
    console.log("Assertion 4 Passed: Name input is enabled");
  });
});
```

### Understanding the Matchers:
- `toContain(substring)`: Checks if a string contains a specific substring. Great for titles or URLs that might have dynamic parameters appended to them.
- `toBe(value)`: Checks for strict equality (`===`). We use this heavily for `isDisplayed()` and `isEnabled()` which return exact boolean `true`/`false`.
- `toEqual(value)`: Checks for deep equality. While `toBe()` works fine for strings, `toEqual()` is generally preferred when comparing exact text payloads.

---

## 3. Test Execution Output

Let's execute this test using our configured test runner. When you type `npm test`, Jest will compile the script and execute the assertions sequentially.

```text
> mcyt-sel-typescript@1.0.0 test
> jest
 PASS  tests/assertions.test.ts (5.811 s)
  Test Assertions using Jest
    √ Should assert element visibility and text on MyCodeYatra Practice (2104 ms)
  console.log
    Navigating to https://practice.mycodeyatra.com/ ...
  console.log
    Assertion 1 Passed: Title is correct
  console.log
    Assertion 2 Passed: Header is visible
  console.log
    Assertion 3 Passed: Header text is 'Automation Practice Form'
  console.log
    Assertion 4 Passed: Name input is enabled
Test Suites: 1 passed, 1 total
Tests:       1 passed, 1 total
Snapshots:   0 total
Time:        6.102 s
Ran all test suites.
```

## Conclusion

Assertions are the core of your automation framework. By combining Selenium WebDriver's element state methods (`isDisplayed()`, `getText()`, `isEnabled()`) with Jest's powerful `expect()` matchers, you can validate complex UI states with absolute precision.

In our next article, we will explore advanced browser debugging by tapping directly into the **Chrome DevTools Protocol (CDP)** using TypeScript!
