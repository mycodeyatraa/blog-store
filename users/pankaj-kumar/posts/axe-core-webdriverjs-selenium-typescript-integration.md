---
title: Scaling Accessibility: Integrating Axe Core with Selenium
date: 02-Jan-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, accessibility, a11y, axe-core, wcag, webdriverjs]
category: Selenium TypeScript
categories: [Selenium TypeScript, Accessibility Testing Automation]
excerpt: >-
  Ditch manual DOM assertions and inject the industry-standard Axe Core engine into Selenium to scan your entire web app for hundreds of WCAG violations instantly.
readTime: 5 min read
---

# Scaling Accessibility: Integrating Axe Core with Selenium

In our last tutorial, we used Selenium `findElements` to manually check for `alt` tags and `aria-labels`. While this works for one or two rules, the WCAG (Web Content Accessibility Guidelines) contains dozens of complex compliance standards. Writing a custom DOM-parsing script for every single rule is not scalable.

To solve this, the industry relies on **Axe Core**, an open-source accessibility engine created by Deque Systems. It is the exact same engine that powers Google Lighthouse and Microsoft's Accessibility Insights.

In this tutorial, we will learn how to integrate Axe Core directly into our Selenium TypeScript framework!

---

## 1. Installing Axe WebdriverJS

Deque provides an official bridge between Axe Core and Selenium WebDriver called `@axe-core/webdriverjs`. 

Install it in your project:

```bash
npm install @axe-core/webdriverjs --save-dev
```

---

## 2. Writing an Axe Core Test Script

Integrating Axe is incredibly simple. You navigate to a page using Selenium, pass the `WebDriver` instance to `AxeBuilder`, and call `.analyze()`. Axe will inject its JavaScript engine into the browser, scan the entire DOM, and return a JSON object detailing every single accessibility violation!

Create `tests/axe_integration.test.ts`:

```typescript
import { Builder, WebDriver } from "selenium-webdriver";
import AxeBuilder from "@axe-core/webdriverjs";
describe("Phase 9 - Accessibility Testing: Axe Core Integration", () => {
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
  it("Should perform a full page accessibility scan using Axe Core", async () => {
    console.log("[Accessibility] Injecting Axe Core into the browser...");
    // Pass the Selenium WebDriver instance to Axe
    const axeBuilder = new AxeBuilder(driver);
    // Analyze the current DOM state
    const results = await axeBuilder.analyze();
    console.log(`[Accessibility] Scan complete. Found ${results.violations.length} violations.`);
    if (results.violations.length > 0) {
      console.error("[Accessibility] Violations details:");
      results.violations.forEach((violation) => {
        console.error(`- Rule: ${violation.id} | Help: ${violation.help}`);
        console.error(`  Affected Nodes: ${violation.nodes.length}`);
      });
    }
    // Assert that there are zero WCAG violations on the page!
    expect(results.violations.length).toBe(0);
  });
});
```

---

## 3. Test Execution Output

Run the Axe suite:

```bash
> ENV_NAME=qa jest tests/axe_integration.test.ts
```

Output:

```text
 FAIL  tests/axe_integration.test.ts
  Phase 9 - Accessibility Testing: Axe Core Integration
    × Should perform a full page accessibility scan using Axe Core (1500 ms)
  console.log
    [Accessibility] Injecting Axe Core into the browser...
    [Accessibility] Scan complete. Found 1 violations.
  console.error
    [Accessibility] Violations details:
    - Rule: color-contrast | Help: Elements must have sufficient color contrast
      Affected Nodes: 1
```

Boom! With just three lines of code, Axe Core audited our entire webpage and found a WCAG color contrast violation that would have taken hours to code manually.

## Conclusion

Integrating Axe Core is the absolute fastest way to bring world-class accessibility testing to your Selenium framework. 

However, Axe scans the entire page indiscriminately. What if we only want to scan a specific modal, or exclude a legacy widget we know is broken? 

In our next tutorial, we will explore **ARIA Validation**, diving deeper into targeted component scanning and configuring Axe to look for specific ARIA (Accessible Rich Internet Applications) tag compliance!
