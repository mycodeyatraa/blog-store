---
title: "Device Emulation (Viewport & User Agent configuration)"
date: "2025-03-30"
description: "Learn how to emulate mobile devices in Playwright TypeScript by configuring viewports, user agents, device scale factors, and utilizing pre-packaged device configurations."
tags: ["Playwright", "TypeScript", "Mobile Emulation", "Viewport", "User Agent"]
---

Welcome to Blog 47 of the **Playwright TypeScript Mastery Series**!

To ensure high-quality user experiences across mobile devices, modern web automation frameworks must support viewport and user agent emulation. Rather than running tests on expensive physical mobile device grids, you can emulate these settings directly in Chromium, WebKit, or Firefox.

Today, we kick off **Phase 7: Mobile Web Emulation & Visual Testing**. We will learn how to configure device profiles (including custom viewport sizes and user agents) in Playwright.

---

### Playwright Device Library

Playwright provides a built-in dictionary of common mobile device configurations (`devices`). Each device configuration includes:
1. **userAgent**: The exact user-agent string used to trigger mobile web server rendering.
2. **viewport**: Default screen width and height.
3. **deviceScaleFactor**: Pixel density scale factor.
4. **isMobile**: Boolean instructing the browser whether to enable mobile touch events.
5. **hasTouch**: Boolean instructing the browser whether to support touch gestures.

---

### Step 1: Implementing the Device Emulation Spec

Create a test file `tests/blog47_device_emulation.spec.ts` showing how to dynamically apply viewport and device configurations to a test page:

```typescript
import { test, expect, devices } from '@playwright/test';
 
test.describe('Blog 47: Mobile Device Emulation in Playwright', () => {
 
  test('Emulate iPhone 12 viewport and user agent', async ({ page }) => {
    const iPhone12 = devices['iPhone 12'];
    
    // Emulate iPhone 12 viewport size
    await page.setViewportSize(iPhone12.viewport);
    
    // Navigate to dummy page displaying resolution and user agent
    await page.goto('data:text/html,<html><body><h1 id="screen-info"></h1><script>document.getElementById("screen-info").innerText = window.innerWidth;</script></body></html>');
    
    const infoText = await page.textContent('#screen-info');
    console.log('[Device Emulation] Page Viewport Width:', infoText);
    
    // Viewport width for iPhone 12 is 390
    expect(infoText).toBe('390');
  });
 
});
```

---

### Step 2: Run the Test

Execute the test via Playwright CLI:

```
npx playwright test tests/blog47_device_emulation.spec.ts
```

**Output:**

```
Running 1 test using 1 worker
 
[Device Emulation] Page Viewport Width: 390
  ✓  1 tests/blog47_device_emulation.spec.ts:5:7 › Blog 47: Mobile Device Emulation in Playwright › Emulate iPhone 12 viewport and user agent (295ms)
 
  1 passed (1.9s)
```

In the next blog, we will build upon this by learning how to automate **Mobile Touch Events & Gestures** like taps, swipes, and scrolls!
