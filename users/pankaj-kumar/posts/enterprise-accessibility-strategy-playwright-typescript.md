---
title: Building an Enterprise Accessibility Strategy in Playwright TypeScript
date: 15-Apr-2025
lastUpdated: 15-Apr-2025
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: ["playwright", "typescript", "axe-core", "accessibility", "enterprise", "fixtures"]
category: UI Automation
categories: ["UI Automation", "Playwright", "TypeScript", "Accessibility", "Enterprise"]
excerpt: >-
  Architect a scalable, global accessibility testing framework using Playwright custom fixtures and axe-core to enforce standardized enterprise compliance.
readTime: 5 min read
---

When scaling UI automation across multiple teams within an enterprise, copying and pasting `AxeBuilder` configurations into every test file becomes unmanageable. If a new company-wide compliance standard is introduced, developers would have to update hundreds of files.

In this tutorial, we will use Playwright's powerful **Custom Fixtures** to centralize our accessibility strategy, ensuring all teams adhere to a strict, standardized enterprise policy.

### Centralizing Axe Configurations using Custom Fixtures

By extending Playwright's base `test` object, we can inject a pre-configured `makeAxeBuilder` function directly into every test's runtime context. This function acts as the single source of truth for your company's accessibility standards.

**File:** `tests/enterprise-accessibility.spec.ts`

```typescript
import { test as base, expect } from '@playwright/test';
import AxeBuilder from '@axe-core/playwright';
 
// 1. Define custom fixtures for enterprise testing
type EnterpriseFixtures = {
    makeAxeBuilder: () => AxeBuilder;
};
 
// 2. Extend the base test to automatically inject pre-configured accessibility tools
export const test = base.extend<EnterpriseFixtures>({
    makeAxeBuilder: async ({ page }, use) => {
        // Create an enterprise-standardized axe builder that all teams must use
        const makeAxeBuilder = () => new AxeBuilder({ page })
            // Enterprise standard: Always check for WCAG 2.1 AA compliance
            .withTags(['wcag2a', 'wcag2aa', 'wcag21a', 'wcag21aa'])
            // Enterprise legacy exception: Disable color contrast on legacy components
            .disableRules(['color-contrast']);
 
        await use(makeAxeBuilder);
    }
});
 
// 3. Utilize the custom fixture in a standard test
test.describe('Enterprise Accessibility Strategy', () => {
 
    test('should validate dashboard with enterprise compliance rules', async ({ page, makeAxeBuilder }) => {
        await page.goto('https://practice.mycodeyatra.com/');
        await page.waitForLoadState('networkidle');
 
        // Use the injected builder directly without configuring it!
        const results = await makeAxeBuilder().analyze();
 
        console.log(`[Enterprise Scan] Violations found: ${results.violations.length}`);
 
        if (results.violations.length > 0) {
            results.violations.forEach(v => {
                console.log(`Violation: ${v.id} - ${v.description}`);
            });
        }
 
        // Assert compliance
        expect(results.violations.length).toBe(0);
    });
 
});
```

### Execution Output

By invoking the custom fixture, the test safely overrides the default rules engine and executes the enterprise policy. Because we globally disabled the `color-contrast` rule (a common pattern for ignoring known legacy tech-debt), the scan successfully passes!

```
Running 1 test using 1 worker
 
[Enterprise Scan] Violations found: 0
  ✓  1 tests\enterprise-accessibility.spec.ts:26:5 › Enterprise Accessibility Strategy › should validate dashboard with enterprise compliance rules (5.4s)
 
  1 passed (6.9s)
```

### Summary

Custom fixtures transform accessibility testing from a localized chore into a scalable architecture. By funneling all scans through a centralized `makeAxeBuilder`, SDET architects can globally enforce WCAG compliance standards, manage exclusions for known legacy defects, and seamlessly roll out accessibility updates across the entire engineering organization without breaking downstream pipelines!
