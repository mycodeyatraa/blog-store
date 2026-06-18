---
title: Designing for Everyone: An Introduction to Web Accessibility Testing
date: 01-Jan-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, accessibility, a11y, wcag, screen-reader]
category: Selenium TypeScript
categories: [Selenium TypeScript, Accessibility Testing Automation]
excerpt: >-
  Step into Phase 9 of our curriculum and learn how to use standard Selenium commands to assert basic WCAG compliance for screen readers.
readTime: 4 min read
---

# Designing for Everyone: An Introduction to Web Accessibility Testing

Welcome to Phase 9 of our Selenium TypeScript curriculum! Over the next few tutorials, we will explore an entirely new paradigm of Quality Engineering: **Web Accessibility Testing (a11y)**.

Have you ever wondered how a person who is entirely blind navigates Amazon to buy a product? They use specialized software called a **Screen Reader** (like JAWS, NVDA, or Apple VoiceOver). A screen reader parses the HTML Document Object Model (DOM) and reads it out loud to the user.

If a developer builds a button using a `<div>` tag with CSS styling and no descriptive text, a sighted user can easily click it, but a blind user will have absolutely no idea it exists.

Accessibility testing ensures our web applications are usable by everyone, regardless of disability. In many countries (including the US and EU), strict compliance with the **WCAG** (Web Content Accessibility Guidelines) is mandated by law.

---

## 1. Manual DOM Assertions for Accessibility

Before we reach for advanced tools and AI scanners, it is crucial to understand the foundational rules of HTML accessibility. Two of the most common accessibility violations are:
1. Images missing alternative text (`alt` attributes).
2. Buttons missing accessible names (text or `aria-label` attributes).

We can automate checks for these using standard Selenium `WebDriver` commands!

Create `tests/accessibility_basics.test.ts`:

```typescript
import { Builder, WebDriver, By } from "selenium-webdriver";
describe("Phase 9 - Accessibility Testing: Basics and Manual DOM Assertions", () => {
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
  it("Should verify that all <img> tags have a descriptive 'alt' attribute", async () => {
    console.log("[Accessibility] Scanning DOM for image tags...");
    const images = await driver.findElements(By.tagName("img"));
    console.log(`[Accessibility] Found ${images.length} images. Checking alt text...`);
    for (const img of images) {
      const altText = await img.getAttribute("alt");
      // Accessibility Rule: Images MUST have an alt attribute. 
      // If the image is purely decorative, the alt attribute must exist but be empty (alt="").
      expect(altText).not.toBeNull(); 
      // Screen readers announce images. We don't want them saying "Image image"
      if (altText) {
        expect(altText.toLowerCase()).not.toMatch(/^(image|picture)$/);
      }
    }
  });
  it("Should verify that all interactive buttons have accessible text or aria-labels", async () => {
    console.log("[Accessibility] Scanning DOM for button tags...");
    const buttons = await driver.findElements(By.tagName("button"));
    console.log(`[Accessibility] Found ${buttons.length} buttons. Checking accessible names...`);
    for (const btn of buttons) {
      const innerText = await btn.getText();
      const ariaLabel = await btn.getAttribute("aria-label");
      // Accessibility Rule: Buttons MUST have screen-reader accessible text.
      // This can come from innerText or an aria-label.
      const hasAccessibleName = (innerText && innerText.trim().length > 0) || (ariaLabel && ariaLabel.trim().length > 0);
      expect(hasAccessibleName).toBe(true);
    }
  });
});
```

---

## 2. Test Execution Output

Run the basics suite:

```bash
> ENV_NAME=qa jest tests/accessibility_basics.test.ts
```

Output:

```text
 PASS  tests/accessibility_basics.test.ts
  Phase 9 - Accessibility Testing: Basics and Manual DOM Assertions
    √ Should verify that all <img> tags have a descriptive 'alt' attribute (210 ms)
    √ Should verify that all interactive buttons have accessible text or aria-labels (145 ms)
  console.log
    [Accessibility] Scanning DOM for image tags...
    [Accessibility] Found 14 images. Checking alt text...
    [Accessibility] Scanning DOM for button tags...
    [Accessibility] Found 8 buttons. Checking accessible names...
```

If any developer forgets an `alt` attribute or builds a button consisting solely of a FontAwesome icon without an `aria-label`, this test will instantly fail!

## Conclusion

Manual DOM assertions are a great way to enforce specific rules on specific components, but writing `findElements` loops for every single WCAG guideline (of which there are hundreds) is impossible.

In our next tutorial, we will supercharge our accessibility testing by integrating **Axe Core**, the industry standard accessibility engine used by Microsoft, Google, and Deque Systems!
