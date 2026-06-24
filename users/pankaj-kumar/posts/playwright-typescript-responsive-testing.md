---
title: Responsive Testing with Playwright TypeScript: Viewports and Devices
date: 06-Apr-2025
lastUpdated: 06-Apr-2025
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: ["playwright", "typescript", "responsive testing", "mobile"]
category: UI Automation
categories: ["UI Automation", "Playwright", "TypeScript"]
excerpt: >-
  Learn how to automate responsive layout testing across Desktop, Tablet, and Mobile using Playwright viewports.
readTime: 4 min read
---

Responsive design ensures your application looks and functions well across varying screen sizes and devices. With Playwright, performing responsive testing is incredibly straightforward because it natively supports browser contexts with customizable viewports and device emulations.

In this tutorial, we will learn how to automate responsive tests across Mobile, Tablet, and Desktop screen sizes using Playwright TypeScript.

### Configuring Viewports in Playwright

Playwright allows you to override context options for a specific test or an entire test suite using `test.use()`. Let's look at an example targeting the MyCodeYatra practice application.

**File:** `tests/responsive-testing.spec.ts`

```typescript
import { test, expect } from '@playwright/test';
 
// Desktop Viewport
test.describe('Responsive Testing - Desktop', () => {
    test.use({ viewport: { width: 1920, height: 1080 } });
 
    test('should display desktop navigation menu', async ({ page }) => {
        await page.goto('https://practice.mycodeyatra.com/');
 
        // Desktop elements should be visible
        const header = page.locator('header');
        await expect(header).toBeVisible();
        // Additional Desktop specific checks...
    });
});
 
// Tablet Viewport
test.describe('Responsive Testing - Tablet', () => {
    test.use({ viewport: { width: 768, height: 1024 } });
 
    test('should adapt layout for tablet view', async ({ page }) => {
        await page.goto('https://practice.mycodeyatra.com/');
 
        // Validate layout changes
        const header = page.locator('header');
        await expect(header).toBeVisible();
    });
});
 
// Mobile Viewport
test.describe('Responsive Testing - Mobile', () => {
    test.use({ viewport: { width: 375, height: 667 }, hasTouch: true });
 
    test('should show hamburger menu on mobile', async ({ page }) => {
        await page.goto('https://practice.mycodeyatra.com/');
 
        // Mobile specific validation
        // Hamburger menu icon should be visible on mobile screens
        // const hamburger = page.locator('.mobile-menu-btn');
        // await expect(hamburger).toBeVisible();
    });
});
```

### Explanation

1. **`test.use({ viewport: { ... } })`**: This overrides the default Playwright viewport dimensions for all tests within the `test.describe` block.
2. **`hasTouch: true`**: When emulating mobile devices, we can enable touch screen semantics, which is important if your app relies on touch events rather than clicks.
3. **Assertions**: Depending on the viewport, different elements (like a hamburger menu instead of a full navigation bar) will be visible. We use `expect(locator).toBeVisible()` to validate the DOM correctly represents the viewport state.

### Using Pre-defined Devices

Instead of manually typing resolutions, Playwright also ships with an extensive list of pre-configured devices:

```typescript
import { test, devices } from '@playwright/test';
 
test.use(devices['iPhone 13 Pro']);
 
test('Mobile layout test', async ({ page }) => {
    // Context is now configured as an iPhone 13 Pro
    await page.goto('https://practice.mycodeyatra.com/');
    // ...
});
```

Responsive testing is vital in modern web applications, and Playwright's contextual isolation makes testing every breakpoint fast and reliable!
