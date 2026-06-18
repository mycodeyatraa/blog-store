---
title: Math in the DOM: Automating Color Contrast Testing
date: 04-Jan-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, accessibility, color-contrast, axe-core, wcag, css]
category: Selenium TypeScript
categories: [Selenium TypeScript, Accessibility Testing Automation]
excerpt: >-
  Discover how to configure Axe Core to extract CSS properties and calculate foreground-to-background contrast ratios, ensuring your UI meets strict WCAG AA thresholds for low-vision users.
readTime: 4 min read
---

# Math in the DOM: Automating Color Contrast Testing

Accessibility is not just about screen readers. A massive portion of users who rely on accessibility guidelines are sighted users who suffer from low vision or color blindness. 

If a UI designer creates a beautiful minimalist page featuring light grey text on a white background, users with low vision will not be able to read the content. 

To prevent this, the **WCAG 2.0 AA standard** enforces strict mathematical formulas for color contrast:
- Normal text MUST have a contrast ratio of at least **4.5:1** against its background.
- Large text (18pt or 14pt bold) MUST have a contrast ratio of at least **3.0:1**.

Calculating this manually is tedious. Let's configure Axe Core to do the math for us!

---

## 1. Targeted Rule Execution with Axe

While Axe evaluates contrast by default during a full scan, we can build a test that strictly focuses on color math by passing the `color-contrast` rule ID to the `.withRules()` method.

Create `tests/color_contrast.test.ts`:

```typescript
import { Builder, WebDriver } from "selenium-webdriver";
import AxeBuilder from "@axe-core/webdriverjs";
describe("Phase 9 - Accessibility Testing: Color Contrast Evaluation", () => {
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
  it("Should calculate foreground-to-background contrast ratios and enforce WCAG thresholds", async () => {
    console.log("[Accessibility] Executing targeted Color Contrast analysis...");
    // We configure Axe to ONLY run the color-contrast rule
    const axeBuilder = new AxeBuilder(driver)
      .withRules(['color-contrast']); 
    const results = await axeBuilder.analyze();
    console.log(`[Accessibility] Scan complete. Found ${results.violations.length} Color Contrast violations.`);
    if (results.violations.length > 0) {
      const colorViolation = results.violations[0];
      console.error(`[Accessibility] Violation: ${colorViolation.help}`);
      // Axe Core is incredible. It literally gives you the mathematical ratio it calculated!
      colorViolation.nodes.forEach((node) => {
        console.error(`- Node: ${node.html}`);
        console.error(`- Math: ${node.failureSummary}`);
      });
    }
    // Assert that the site passes contrast checks
    expect(results.violations.length).toBe(0);
  });
});
```

---

## 2. Test Execution Output

Run the contrast suite against a site with poor color choices:

```bash
> ENV_NAME=qa jest tests/color_contrast.test.ts
```

Output:

```text
 FAIL  tests/color_contrast.test.ts
  Phase 9 - Accessibility Testing: Color Contrast Evaluation
    × Should calculate foreground-to-background contrast ratios (1100 ms)
  console.log
    [Accessibility] Executing targeted Color Contrast analysis...
    [Accessibility] Scan complete. Found 1 Color Contrast violations.
  console.error
    [Accessibility] Violation: Elements must have sufficient color contrast
    - Node: <button style='background-color: #d3d3d3; color: #ffffff;'>Submit</button>
    - Math: Fix any of the following:
      Element has insufficient color contrast of 1.48 (foreground color: #ffffff, background color: #d3d3d3, font size: 12.0pt (16px), font weight: normal). Expected contrast ratio of 4.5:1
```

Look at that output! Axe didn't just tell us the contrast failed; it calculated the exact ratio (`1.48:1`), identified the specific hex codes (`#ffffff` on `#d3d3d3`), parsed the CSS font-size (`16px`), and printed the exact WCAG expectation (`4.5:1`). 

This level of detail makes it incredibly easy to open a bug ticket and tell your UI developer exactly what needs fixing.

## Conclusion

By leveraging Axe Core's mathematical contrast engine, we can ensure our application is perfectly legible for all users without ever having to manually calculate hex color diffs.

However, printing JSON logs to the console isn't a great experience for project managers or stakeholders who want to review our accessibility compliance over time. 

In our next tutorial, we will learn how to generate beautiful HTML **Accessibility Reports**!
