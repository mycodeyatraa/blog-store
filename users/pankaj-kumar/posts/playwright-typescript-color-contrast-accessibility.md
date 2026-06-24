---
title: Automated Color Contrast Testing in Playwright TypeScript
date: 12-Apr-2025
lastUpdated: 12-Apr-2025
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: ["playwright", "typescript", "axe-core", "accessibility", "color-contrast"]
category: UI Automation
categories: ["UI Automation", "Playwright", "TypeScript", "Accessibility"]
excerpt: >-
  Leverage axe-core to mathematically validate UI color contrast ratios automatically within your test suites.
readTime: 5 min read
---

Color contrast is one of the most common accessibility failures on the web. Low contrast text makes applications difficult to use for people with visual impairments or simply users under bright sunlight.

In this tutorial, we will use `@axe-core/playwright` to isolate color contrast checks on a web application, identifying exactly which elements fail to meet the WCAG 2 AA contrast ratio threshold of 4.5:1.

### Writing the Contrast Test

Using `AxeBuilder`, we can specifically target the `cat.color` tag. This tag exclusively filters the rules engine to evaluate contrast.

**File:** `tests/color-contrast.spec.ts`

```typescript
import { test, expect } from '@playwright/test';
import AxeBuilder from '@axe-core/playwright';
 
test.describe('Color Contrast Testing with Axe-Core', () => {
 
    test('should validate color contrast across the entire page', async ({ page }) => {
        // Navigate to the target page
        await page.goto('https://practice.mycodeyatra.com/');
        await page.waitForLoadState('networkidle');
 
        // Analyze the page but filter ONLY for color contrast rules
        // 'cat.color' groups rules related to color and contrast accessibility
        const results = await new AxeBuilder({ page })
            .withTags(['cat.color'])
            .analyze();
 
        console.log(`[Color Contrast Scan] Violations found: ${results.violations.length}`);
 
        // Print the failing nodes to help developers locate the low-contrast elements
        if (results.violations.length > 0) {
            results.violations.forEach(violation => {
                console.log(`\nRule Violated: ${violation.id}`);
                console.log(`Description: ${violation.description}`);
                console.log(`Impact: ${violation.impact}`);
 
                violation.nodes.forEach((node, index) => {
                    console.log(`  Issue ${index + 1}:`);
                    console.log(`  Selector: ${node.target.join(', ')}`);
                    console.log(`  Snippet: ${node.html}`);
                    console.log(`  Fix Summary: ${node.failureSummary}`);
                });
            });
        }
 
        // We expect no color contrast violations
        expect(results.violations).toEqual([]);
    });
 
});
```

### Execution Output

When we run this test against the practice application, we immediately catch several real-world failures! Here is the captured output directly from the test runner:

```
Rule Violated: color-contrast
Description: Ensures the contrast between foreground and background colors meets WCAG 2 AA contrast ratio thresholds
Impact: serious
  Issue 1:
  Selector: .card.hover-card.glass:nth-child(42) > div:nth-child(2) > button
  Snippet: <button style="...">
  Fix Summary: Fix any of the following:
  Element has insufficient color contrast of 4.23 (foreground color: #8b5cf6, background color: #ffffff, font size: 9.6pt (12.8px), font weight: bold). Expected contrast ratio of 4.5:1
 
  Issue 2:
  Selector: .card.hover-card.glass:nth-child(46) > div:nth-child(2) > button
  Snippet: <button style="...">
  Fix Summary: Fix any of the following:
  Element has insufficient color contrast of 3.85 (foreground color: #0288d1, background color: #ffffff, font size: 9.6pt (12.8px), font weight: bold). Expected contrast ratio of 4.5:1
 
  1 failed
    tests\color-contrast.spec.ts:6:9 › Color Contrast Testing with Axe-Core › should validate color contrast across the entire page
```

### Summary

As seen in the test output, `axe-core` automatically calculates the computed styles. It knows that foreground `#8b5cf6` against background `#ffffff` only achieves a 4.23 ratio, mathematically failing the required 4.5:1! This actionable feedback allows developers to fix CSS instantly without guessing.
