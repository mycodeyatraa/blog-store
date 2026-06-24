---
title: Accessibility Testing Basics in Playwright TypeScript
date: 09-Apr-2025
lastUpdated: 09-Apr-2025
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: ["playwright", "typescript", "accessibility", "axe-core"]
category: UI Automation
categories: ["UI Automation", "Playwright", "TypeScript", "Accessibility"]
excerpt: >-
  Learn how to use axe-core to find WCAG accessibility violations directly within your Playwright test suites.
readTime: 4 min read
---

Accessibility (a11y) testing ensures that your web application is usable by everyone, including people with disabilities. Automated accessibility testing can catch common issues like missing ARIA attributes or poor color contrast before they reach production.

In this tutorial, we will learn how to integrate the industry-standard `axe-core` library with Playwright TypeScript to automatically scan pages for accessibility violations.

### Prerequisites

To get started, install the `@axe-core/playwright` package:

```bash
npm install -D @axe-core/playwright
```

### Writing the Accessibility Test

Using `axe-core` in Playwright is extremely straightforward. You navigate to the page, wait for it to load, and then execute the `AxeBuilder` analysis.

**File:** `tests/accessibility.spec.ts`

```typescript
import { test, expect } from '@playwright/test';
import AxeBuilder from '@axe-core/playwright';
 
test.describe('Accessibility Testing Basics', () => {
 
    test('should not have any automatically detectable accessibility issues', async ({ page }) => {
        // Navigate to the target page
        await page.goto('https://practice.mycodeyatra.com/');
 
        // Wait for the page to be fully rendered
        await page.waitForLoadState('networkidle');
 
        // Run axe-core analysis on the page
        const accessibilityScanResults = await new AxeBuilder({ page }).analyze();
 
        // Print results explicitly to capture execution outputs
        if (accessibilityScanResults.violations.length > 0) {
            console.log('Accessibility Violations Found:');
            accessibilityScanResults.violations.forEach(v => {
                console.log(`- Rule: ${v.id}`);
                console.log(`  Description: ${v.description}`);
                console.log(`  Impact: ${v.impact}`);
                console.log(`  Nodes affected: ${v.nodes.length}`);
            });
        } else {
            console.log('No accessibility violations found! Perfect!');
        }
 
        // Assert that there are no violations
        expect(accessibilityScanResults.violations).toEqual([]);
    });
 
});
```

### Execution Output

When running the test against our practice application, it actually caught a real-world contrast issue! Here is a snippet of the test output capturing the failure:

```
Accessibility Violations Found:
- Rule: color-contrast
  Description: Ensures the contrast between foreground and background colors meets WCAG 2 AA contrast ratio thresholds
  Impact: serious
  Nodes affected: 46
 
  1 failed
    tests/accessibility.spec.ts:6:9 › Accessibility Testing Basics › should not have any automatically detectable accessibility issues
```

The error context explicitly tells us:
`Element has insufficient color contrast of 4.23 (foreground color: #8b5cf6, background color: #ffffff... Expected contrast ratio of 4.5:1`

### Summary

With just a few lines of code, `AxeBuilder` scanned the entire DOM and found exactly which elements failed the WCAG 2 AA guidelines for contrast ratios. By running this automatically in CI/CD pipelines, teams can enforce accessibility standards effortlessly!
