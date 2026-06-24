---
title: Mobile Web Device Emulation in Playwright TypeScript
date: 07-Apr-2025
lastUpdated: 07-Apr-2025
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: ["playwright", "typescript", "mobile emulation", "testing"]
category: UI Automation
categories: ["UI Automation", "Playwright", "TypeScript", "Mobile"]
excerpt: >-
  Simulate iOS and Android devices, set custom user agents, and enable touch events for mobile web testing in Playwright.
readTime: 4 min read
---

Testing how your application behaves on mobile devices is critical since the majority of global web traffic comes from smartphones. Playwright makes mobile web testing incredibly easy by providing built-in device emulations.

In this tutorial, we will explore how to simulate iOS and Android devices, set custom user agents, and enable touch events for testing in Playwright TypeScript.

### Built-in Device Emulation

Playwright comes with a dictionary of pre-configured devices. This includes accurate user agents, screen sizes, and pixel ratios for popular phones and tablets.

**File:** `tests/mobile-emulation.spec.ts`

```typescript
import { test, expect, devices } from '@playwright/test';
 
// Import pre-configured devices
const iPhone13 = devices['iPhone 13 Pro'];
const pixel5 = devices['Pixel 5'];
 
test.describe('Mobile Web Device Emulation', () => {
 
    test.describe('iPhone 13 Pro Emulation', () => {
        // Apply the device configuration to the context
        test.use({ ...iPhone13 });
 
        test('should render properly on iPhone 13 Pro', async ({ page }) => {
            await page.goto('https://practice.mycodeyatra.com/');
 
            // Validate visibility of elements meant for mobile view
            const header = page.locator('header');
            await expect(header).toBeVisible();
 
            console.log(`Viewport Size: ${page.viewportSize()?.width}x${page.viewportSize()?.height}`);
        });
    });
 
    test.describe('Pixel 5 Emulation', () => {
        // Apply the Android device configuration
        test.use({ ...pixel5 });
 
        test('should render properly on Pixel 5 Android', async ({ page }) => {
            await page.goto('https://practice.mycodeyatra.com/');
 
            // Check that the user agent is injected correctly
            const userAgent = await page.evaluate(() => navigator.userAgent);
            expect(userAgent).toContain('Android');
        });
    });
 
});
```

### Custom Mobile Emulation Configurations

Sometimes, you need to test a screen resolution that is not included in the default device list, or you want fine-grained control over touch emulation. You can construct a custom context using `test.use()`:

```typescript
    test.describe('Custom Mobile Emulation', () => {
        test.use({
            viewport: { width: 414, height: 896 },
            userAgent: 'Mozilla/5.0 (iPhone; CPU iPhone OS 15_0 like Mac OS X)',
            deviceScaleFactor: 3,
            isMobile: true,
            hasTouch: true
        });
 
        test('should allow custom user agent and touch emulation', async ({ page }) => {
            await page.goto('https://practice.mycodeyatra.com/');
 
            // Validate that touch events are enabled
            const isTouchSupported = await page.evaluate(() => 'ontouchstart' in window);
            expect(isTouchSupported).toBeTruthy();
        });
    });
```

### Explanation

1. **`devices['Device Name']`**: Retrieves the configuration object for a specific device. You can spread (`...`) this object inside `test.use()` to apply the settings.
2. **`userAgent`**: Mocking the user agent ensures that backend servers and frontend frameworks serve the correct mobile assets to the browser.
3. **`isMobile` & `hasTouch`**: These flags enable meta-viewport behavior and ensure touch events (like swipe and tap) work as expected rather than treating inputs strictly as mouse clicks.

Playwright simplifies mobile web testing by allowing you to test mobile behaviors locally without the overhead of physical devices or heavy emulators!
