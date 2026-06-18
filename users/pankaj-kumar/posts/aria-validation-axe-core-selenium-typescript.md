---
title: Precision Scans: Targeted ARIA Validation with Axe
date: 03-Jan-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, accessibility, aria, axe-core, wcag, ui-automation]
category: Selenium TypeScript
categories: [Selenium TypeScript, Accessibility Testing Automation]
excerpt: >-
  Learn how to configure the AxeBuilder API to isolate specific UI components and execute targeted WCAG ARIA validations to ensure screen reader compliance.
readTime: 4 min read
---

# Precision Scans: Targeted ARIA Validation with Axe

In our previous tutorial, we ran Axe Core against the entire DOM. While this is fantastic for catching broad issues like color contrast or missing `alt` attributes, it can be overwhelming when you only want to test a newly developed UI component.

Furthermore, complex dynamic components—like modal dialogs, auto-completes, or tab panels—require advanced **ARIA (Accessible Rich Internet Applications)** attributes to inform screen readers of their state (e.g., `aria-expanded="true"`, `role="dialog"`). 

In this tutorial, we will learn how to configure Axe Core to target specific DOM selectors and run specific ARIA rule sets.

---

## 1. The Power of `AxeBuilder` Configuration

The `AxeBuilder` class provides a fluent API that allows you to configure exactly *what* elements to scan and *which* rules to evaluate.

* `.include('css-selector')`: Only scans the DOM inside the matched selector.
* `.exclude('css-selector')`: Skips scanning a known broken legacy component.
* `.withTags(['wcag2aa'])`: Only runs rules that belong to the WCAG 2.0 AA standard.
* `.withRules(['rule-id'])`: Runs *only* the specific rules you provide.

Create `tests/aria_validation.test.ts`:

```typescript
import { Builder, WebDriver } from "selenium-webdriver";
import AxeBuilder from "@axe-core/webdriverjs";
describe("Phase 9 - Accessibility Testing: Targeted ARIA Validation", () => {
  let driver: WebDriver;
  const TARGET_URL = "https://practice.mycodeyatra.com/";
  beforeAll(async () => {
    driver = await new Builder().forBrowser("chrome").build();
    await driver.get(TARGET_URL);
  });
  afterAll(async () => {
    if (driver) {
      await driver.quit();
    }
  });
  it("Should target a specific component (e.g., a modal) and validate its ARIA attributes", async () => {
    console.log("[Accessibility] Initializing targeted Axe Core scan...");
    // We construct a highly targeted AxeBuilder instance
    const axeBuilder = new AxeBuilder(driver)
      .include('.modal-dialog') // 1. ONLY scan the modal! Ignore the rest of the page.
      .withTags(['wcag2a', 'wcag2aa']) // 2. Filter by specific WCAG rule sets
      .withRules(['aria-roles', 'aria-valid-attr', 'aria-dialog-name']); // 3. Only run ARIA specific rules!
    const results = await axeBuilder.analyze();
    console.log(`[Accessibility] Targeted scan complete. Found ${results.violations.length} ARIA violations.`);
    // Log the successful validations
    results.passes.forEach((pass) => {
      console.log(`[Accessibility] PASS: ${pass.id} - ${pass.description}`);
    });
    // Assert that the specific modal component is 100% ARIA compliant
    expect(results.violations.length).toBe(0);
  });
});
```

---

## 2. Test Execution Output

Run the targeted ARIA suite:

```bash
> ENV_NAME=qa jest tests/aria_validation.test.ts
```

Output:

```text
 PASS  tests/aria_validation.test.ts
  Phase 9 - Accessibility Testing: Targeted ARIA Validation
    √ Should target a specific component (e.g., a modal) and validate its ARIA attributes (1200 ms)
  console.log
    [Accessibility] Initializing targeted Axe Core scan...
    [Accessibility] Targeted scan complete. Found 0 ARIA violations.
    [Accessibility] PASS: aria-roles - ARIA roles used must conform to valid values
    [Accessibility] PASS: aria-dialog-name - Ensures every ARIA dialog and alertdialog node has an accessible name
```

By heavily filtering our Axe Core execution, our accessibility tests run incredibly fast and are isolated to the specific UI components our SDET team is currently developing!

## Conclusion

Targeted scanning is essential when integrating accessibility checks into a Component Test environment (like Storybook) or when adding tests to legacy applications where you can't fix every global violation at once.

However, while ARIA roles and DOM structure are critical for screen readers, there is another group of users we must support: users with low vision. 

In our next tutorial, we will dedicate an entire test to **Color Contrast Testing**!
