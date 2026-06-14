---
title: Mastering Forms, Radio Buttons & Checkboxes in Selenium TypeScript
date: 09-Nov-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, forms, checkboxes, radio-buttons]
category: Selenium TypeScript
categories: [Selenium TypeScript, UI Automation, Core Automation]
excerpt: >-
  If you don't check the state of a checkbox before clicking it, you will break your tests. Learn the Golden Rules of safely interacting with forms, radio buttons, and text inputs in Selenium.
readTime: 6 min read
---

# Mastering Forms, Radio Buttons & Checkboxes in Selenium TypeScript

Welcome back to the **Core UI Automation** series! The vast majority of business logic in web applications revolves around submitting data via HTML Forms. 

If you do not handle forms correctly, your tests will fail randomly. For example, if you explicitly tell Selenium to `.click()` a checkbox without verifying its current state, you might accidentally uncheck it!

In this tutorial, we will learn the best practices for handling text inputs, radio buttons, and checkboxes using TypeScript on **[practice.mycodeyatra.com](https://practice.mycodeyatra.com/)**.

---

## 1. Handling Text Inputs safely
Filling out text is straightforward. You locate the element and use `.sendKeys()`.
However, in Enterprise frameworks, it's highly recommended to `.clear()` the input field first, just in case the browser auto-filled some data!

```typescript
const emailInput = await driver.findElement(By.css(".email-field"));
await emailInput.clear();
await emailInput.sendKeys("john@example.com");
```

---

## 2. The Golden Rule of Checkboxes and Radio Buttons
A common mistake junior engineers make is simply locating a checkbox and clicking it:

```typescript
// DANGEROUS! What if it is already checked?
await driver.findElement(By.css("input[value='Reading']")).click();
```

If the checkbox is already checked (perhaps by default, or by a previous test), clicking it will **uncheck** it, causing your assertion to fail!

**The Golden Rule:** Always check the element's state using `.isSelected()` before issuing a click command.

```typescript
const readingCheckbox = await driver.findElement(By.css("input[value='Reading']"));
const isChecked = await readingCheckbox.isSelected();
if (!isChecked) {
  await readingCheckbox.click();
}
```

---

## 3. Writing the Forms Test

Let's combine this logic into a fully executable Jest test. 

Create `tests/forms.test.ts`:

```typescript
import { Builder, WebDriver, By } from "selenium-webdriver";
import "chromedriver";
describe("Core UI Automation - Forms and Checkboxes", () => {
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
  it("Should safely fill forms and toggle checkboxes", async () => {
    console.log("Navigating to https://practice.mycodeyatra.com/ ...");
    await driver.get("https://practice.mycodeyatra.com/");
    // 1. Text Inputs
    const nameInput = await driver.findElement(By.css("#name"));
    await nameInput.clear();
    await nameInput.sendKeys("John Automation");
    console.log("Filled Name Input");
    // 2. Radio Buttons (Safely Select)
    const maleRadioBtn = await driver.findElement(By.css("input[value='Male']"));
    const isMaleSelected = await maleRadioBtn.isSelected();
    if (!isMaleSelected) {
      await maleRadioBtn.click();
      console.log("Clicked Male Radio Button");
    }
    // 3. Checkboxes (Safely Select)
    const readingCheckbox = await driver.findElement(By.css("input[value='Reading']"));
    const isReadingChecked = await readingCheckbox.isSelected();
    if (!isReadingChecked) {
      await readingCheckbox.click();
      console.log("Checked the 'Reading' Checkbox");
    } else {
      console.log("The 'Reading' Checkbox was already checked");
    }
    // 4. Assertions
    expect(await maleRadioBtn.isSelected()).toBe(true);
    expect(await readingCheckbox.isSelected()).toBe(true);
  });
});
```

---

## 4. Test Execution Output

```text
> mcyt-sel-typescript@1.0.0 test
> jest
 PASS  tests/forms.test.ts (6.110 s)
  Core UI Automation - Forms and Checkboxes
    √ Should safely fill forms and toggle checkboxes (3204 ms)
  console.log
    Navigating to https://practice.mycodeyatra.com/ ...
  console.log
    Filled Name Input
  console.log
    Clicked Male Radio Button
  console.log
    Checked the 'Reading' Checkbox
Test Suites: 1 passed, 1 total
Tests:       1 passed, 1 total
Snapshots:   0 total
Time:        6.451 s
Ran all test suites.
```

## Conclusion

Handling forms is easy, but handling them **safely** requires discipline. By always clearing text fields and rigorously verifying the `.isSelected()` state of toggles, your TypeScript automation will be rock solid.

In our next article, we will tackle one of the most frustrating UI elements for beginners: **Handling JavaScript Alerts and Dialogs!**
