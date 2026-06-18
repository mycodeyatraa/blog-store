---
title: Beyond the Console: Generating Beautiful HTML Accessibility Reports
date: 05-Jan-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, accessibility, axe-html-reporter, axe-core, reporting, wcag]
category: Selenium TypeScript
categories: [Selenium TypeScript, Accessibility Testing Automation]
excerpt: >-
  Transform raw Axe Core JSON data into interactive, stakeholder-friendly HTML dashboards using the axe-html-reporter library.
readTime: 4 min read
---

# Beyond the Console: Generating Beautiful HTML Accessibility Reports

When `AxeBuilder.analyze()` executes, it returns a massive JSON object detailing every single accessibility pass, violation, incomplete check, and inapplicable rule found on the page.

While printing `results.violations.length` to the terminal is fine for a CI/CD build gate, it is practically useless for a Product Manager or UI Designer who needs to review the specific issues and prioritize bug fixes.

Stakeholders do not want to read raw JSON. They want a report!

In this tutorial, we will use the `axe-html-reporter` library to transform Axe Core's raw JSON payload into a beautiful, interactive HTML dashboard.

---

## 1. Installing the Reporter

First, we need to install the reporting library:

```bash
npm install axe-html-reporter --save-dev
```

This library takes the exact JSON structure returned by `@axe-core/webdriverjs` and parses it into a standalone HTML file.

---

## 2. Generating the HTML Report

The implementation is incredibly straightforward. Once Axe completes its analysis, you simply pass the `results` object to the `createHtmlReport()` function.

Create `tests/accessibility_reports.test.ts`:

```typescript
import { Builder, WebDriver } from "selenium-webdriver";
import AxeBuilder from "@axe-core/webdriverjs";
import { createHtmlReport } from "axe-html-reporter";
describe("Phase 9 - Accessibility Testing: HTML Reporting", () => {
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
  it("Should generate a beautiful HTML accessibility report from Axe Core results", async () => {
    console.log("[Accessibility] Executing full Axe Core scan for reporting...");
    // 1. Run the Axe scan
    const axeBuilder = new AxeBuilder(driver);
    const results = await axeBuilder.analyze();
    console.log(`[Accessibility] Scan complete. Generating HTML Report...`);
    // 2. Generate the HTML report!
    createHtmlReport({
      results: results,
      options: {
        projectKey: "MyCodeYatra",
        outputDir: "reports/accessibility",
        reportFileName: "a11y-report.html"
      }
    });
    console.log(`[Accessibility] HTML Report successfully saved to: reports/accessibility/a11y-report.html`);
    // 3. Assert on violations to fail the pipeline if necessary
    expect(results.violations.length).toBe(0);
  });
});
```

---

## 3. Test Execution Output

Run the reporter suite:

```bash
> ENV_NAME=qa jest tests/accessibility_reports.test.ts
```

Output:

```text
 FAIL  tests/accessibility_reports.test.ts
  Phase 9 - Accessibility Testing: HTML Reporting
    × Should generate a beautiful HTML accessibility report from Axe Core results (2100 ms)
  console.log
    [Accessibility] Executing full Axe Core scan for reporting...
    [Accessibility] Scan complete. Generating HTML Report...
    [Accessibility] HTML Report successfully saved to: reports/accessibility/a11y-report.html
```

If you navigate to your `reports/accessibility` directory and open `a11y-report.html` in Chrome, you will see a gorgeous dashboard. 

The dashboard includes:
- A high-level summary pie chart (Passes vs Violations).
- Categorized tabs for every WCAG violation.
- The exact DOM node HTML snippets that failed.
- The specific contrast ratio math or missing attributes that triggered the failure.
- Deep links directly to Deque University documentation explaining *why* the rule exists and *how* to fix it!

## Conclusion

Generating accessibility reports bridges the gap between automation engineers and product teams. You can attach this HTML artifact directly to your Jenkins or GitHub Actions build, giving everyone instant visibility into the application's WCAG compliance.

However, writing Selenium scripts just to run Axe can feel heavy if a frontend developer just wants to check a single button they are building locally.

In our next tutorial, we will shift left by exploring **Accessibility CLI Tooling**, allowing developers to run these same Axe scans directly from their terminal without writing a single line of Selenium code!
