---
title: ARIA Validation in Playwright TypeScript using Axe-Core Tags
date: 11-Apr-2025
lastUpdated: 11-Apr-2025
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: ["playwright", "typescript", "axe-core", "accessibility", "aria"]
category: UI Automation
categories: ["UI Automation", "Playwright", "TypeScript", "Accessibility"]
excerpt: >-
  Use axe-core tags to strictly validate ARIA properties and semantic compliance in your automated accessibility tests.
readTime: 4 min read
---

Accessible Rich Internet Applications (ARIA) attributes are crucial for screen readers. However, incorrect ARIA implementations can often be worse than no ARIA at all! 

In this tutorial, we will use `@axe-core/playwright` to specifically isolate and validate ARIA attributes and related WCAG guidelines, ensuring our web application semantics are completely valid.

### Using Axe-Core Tags for ARIA Validation

Instead of disabling rules or excluding elements like we did previously, we can instruct `AxeBuilder` to *only* run rules associated with specific tags.

Axe-core categorizes rules into semantic buckets. For ARIA validation, the `cat.aria` tag is perfectly suited. Furthermore, WCAG tags like `wcag412` (Name, Role, Value) implicitly check semantic constraints.

**File:** `tests/aria-validation.spec.ts`

```typescript
import { test, expect } from '@playwright/test';
import AxeBuilder from '@axe-core/playwright';
 
test.describe('ARIA Validation with Axe-Core', () => {
 
    test('should validate ARIA attributes specifically', async ({ page }) => {
        await page.goto('https://practice.mycodeyatra.com/');
        await page.waitForLoadState('networkidle');
 
        // Analyze the page but filter ONLY for ARIA-related rules using Axe tags
        // 'cat.aria' targets all rules related to ARIA usage
        const results = await new AxeBuilder({ page })
            .withTags(['cat.aria'])
            .analyze();
 
        console.log(`[ARIA Scan] Violations found: ${results.violations.length}`);
 
        if (results.violations.length > 0) {
            results.violations.forEach(v => {
                console.log(`ARIA Violation Rule: ${v.id}`);
                console.log(`Impact: ${v.impact}`);
                console.log(`Description: ${v.description}`);
            });
        }
 
        expect(results.violations).toEqual([]);
    });
 
    test('should validate specific WCAG ARIA guidelines', async ({ page }) => {
        await page.goto('https://practice.mycodeyatra.com/login');
        await page.waitForLoadState('networkidle');
 
        // Targeting specific WCAG tags that enforce ARIA semantics (like wcag412 for Name, Role, Value)
        const results = await new AxeBuilder({ page })
            .withTags(['wcag412', 'wcag131'])
            .analyze();
 
        console.log(`[WCAG ARIA Scan] Violations found: ${results.violations.length}`);
 
        expect(results.violations).toEqual([]);
    });
 
});
```

### Execution Output

When we run these tailored ARIA-specific tests against the practice application, it successfully filters out other generic failures (like contrast issues) and strictly analyzes the DOM semantics:

```
Running 2 tests using 1 worker
 
[ARIA Scan] Violations found: 0
  ✓  1 tests\aria-validation.spec.ts:6:9 › ARIA Validation with Axe-Core › should validate ARIA attributes specifically (2.1s)
[WCAG ARIA Scan] Violations found: 0
  ✓  2 tests\aria-validation.spec.ts:29:9 › ARIA Validation with Axe-Core › should validate specific WCAG ARIA guidelines (2.3s)
 
  2 passed (6.5s)
```

### Summary

Using `.withTags()`, we can construct focused test suites that validate specific criteria like ARIA usage without muddying the waters with unrelated accessibility violations. This approach is highly effective when refactoring semantic HTML layers step-by-step.
