---
title: Advanced Axe Integration in Playwright TypeScript
date: 10-Apr-2025
lastUpdated: 10-Apr-2025
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: ["playwright", "typescript", "axe-core", "accessibility"]
category: UI Automation
categories: ["UI Automation", "Playwright", "TypeScript", "Accessibility"]
excerpt: >-
  Learn how to filter accessibility rules, include specific elements, and exclude legacy components using axe-core.
readTime: 4 min read
---

In our previous tutorial, we learned how to run a basic Axe-core accessibility scan. However, in real-world projects, you may encounter third-party widgets, legacy components, or specific design choices that trigger accessibility violations which you either cannot control or have consciously accepted as exceptions.

In this tutorial, we will learn advanced `@axe-core/playwright` integration techniques: targeting specific elements, excluding problematic DOM nodes, and disabling specific accessibility rules.

### Advanced Axe-Core Integrations

Playwright's `AxeBuilder` offers a fluent API to customize the scope of the accessibility scan.

**File:** `tests/axe-integration.spec.ts`

```typescript
import { test, expect } from '@playwright/test';
import AxeBuilder from '@axe-core/playwright';
 
test.describe('Advanced Axe-Core Integration', () => {
 
    test('should run accessibility tests on a specific element', async ({ page }) => {
        await page.goto('https://practice.mycodeyatra.com/');
        await page.waitForLoadState('networkidle');
 
        // Analyze only the navigation header
        const results = await new AxeBuilder({ page })
            .include('header')
            .analyze();
 
        console.log(`[Include Scope] Accessibility Violations in header: ${results.violations.length}`);
        expect(results.violations).toEqual([]);
    });
 
    test('should exclude specific elements from accessibility scanning', async ({ page }) => {
        await page.goto('https://practice.mycodeyatra.com/');
        await page.waitForLoadState('networkidle');
 
        // Exclude the main cards which are known to have contrast issues
        const results = await new AxeBuilder({ page })
            .exclude('.card')
            .analyze();
 
        console.log(`[Exclude Scope] Accessibility Violations outside of cards: ${results.violations.length}`);
        expect(results.violations).toEqual([]);
    });
 
    test('should disable specific accessibility rules', async ({ page }) => {
        await page.goto('https://practice.mycodeyatra.com/');
        await page.waitForLoadState('networkidle');
 
        // Analyze the whole page, but ignore 'color-contrast' rules entirely
        const results = await new AxeBuilder({ page })
            .disableRules(['color-contrast'])
            .analyze();
 
        console.log(`[Rule Filtering] Violations excluding color-contrast: ${results.violations.length}`);
        expect(results.violations).toEqual([]);
    });
 
});
```

### Execution Output

When we run these tailored tests against the practice application, we can successfully bypass the known contrast issues and achieve a passing test suite!

```
Running 3 tests using 1 worker
 
[Include Scope] Accessibility Violations in header: 0
  ✓  1 tests/axe-integration.spec.ts:6:9 › Advanced Axe-Core Integration › should run accessibility tests on a specific element (2.3s)
[Exclude Scope] Accessibility Violations outside of cards: 0
  ✓  2 tests/axe-integration.spec.ts:20:9 › Advanced Axe-Core Integration › should exclude specific elements from accessibility scanning (2.7s)
[Rule Filtering] Violations excluding color-contrast: 0
  ✓  3 tests/axe-integration.spec.ts:38:9 › Advanced Axe-Core Integration › should disable specific accessibility rules (4.9s)
 
  3 passed (12.1s)
```

### Summary

By using `.include()`, `.exclude()`, and `.disableRules()`, you gain surgical precision over your automated accessibility testing. This allows your team to incrementally adopt accessibility standards without blocking CI/CD pipelines due to legacy debt!
