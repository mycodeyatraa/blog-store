---
title: "Mobile Touch Events & Gestures (tap, swipe, scroll)"
date: "2025-03-31"
description: "Learn how to enable and test mobile touch gestures in Playwright TypeScript, including dispatching taps and managing touch-enabled browsers."
tags: ["Playwright", "TypeScript", "Mobile Emulation", "Touch Gestures", "hasTouch"]
---

Welcome to Blog 48 of the **Playwright TypeScript Mastery Series**!

In mobile web applications, users don't interact via a cursor. They tap, swipe, pinch, and scroll using touch input. Testing these applications requires the test environment to dispatch actual touch events instead of standard mouse clicks.

Today, we will learn how to configure mobile touch events in Playwright, enable touch capabilities in the browser context, and execute tap gestures.

---

### Enabling Touch in Playwright

To test touch gestures, the browser context must be instructed to support touch input. By default, standard desktop contexts disable touch. 

In Playwright, you can enable touch using the `hasTouch` option. When `hasTouch` is set to `true`, the browser will emulate a touch-capable screen and send standard touch event listeners (like `touchstart`, `touchend`, and `touchmove`) to the page.

---

### Step 1: Implementing the Touch Spec

Create a test file `tests/blog48_touch_gestures.spec.ts` showing how to declare `hasTouch: true` and execute touch gestures:

```typescript
import { test, expect } from '@playwright/test';
 
test.describe('Blog 48: Touch Gestures and Mobile Events in Playwright', () => {
  test.use({ hasTouch: true });
 
  test('Perform mobile tap gesture', async ({ page }) => {
    // Navigate to a simple page with a button recording click/tap event
    await page.goto('data:text/html,<html><body><button id="touch-btn" onclick="this.innerText=\'TAPPED\'">Tap Me</button></body></html>');
    
    // Perform tap interaction simulating a mobile touchscreen tap
    await page.tap('#touch-btn');
    
    // Verify changes
    const buttonText = await page.innerText('#touch-btn');
    expect(buttonText).toBe('TAPPED');
    console.log('[Touch Gestures] Mobile tap gesture executed successfully.');
  });
 
});
```

---

### Step 2: Run the Test

Execute the tests using the Playwright CLI:

```
npx playwright test tests/blog48_touch_gestures.spec.ts
```

**Output:**

```
Running 1 test using 1 worker
 
[Touch Gestures] Mobile tap gesture executed successfully.
  ✓  1 tests/blog48_touch_gestures.spec.ts:6:7 › Blog 48: Touch Gestures and Mobile Events in Playwright › Perform mobile tap gesture (280ms)
 
  1 passed (1.5s)
```

In the next blog, we will learn how to configure and test **Geolocation & Timezone Emulation** to simulate user journeys from different locations globally!
