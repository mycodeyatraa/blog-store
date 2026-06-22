---
title: "Handling Inline Frames (iFrames)"
date: "2025-03-01"
description: "iFrames hide elements from normal locators. Learn how Playwright penetrates iFrames seamlessly using frame locators without complex context switching."
tags: ["Playwright", "TypeScript", "iFrame", "FrameLocator"]
---

Welcome to Blog 17 of the **Playwright TypeScript Mastery Series**!

An **iFrame** (Inline Frame) is an HTML document embedded inside another HTML document. They are incredibly common for embedding third-party widgets, like YouTube videos, payment gateways, or customer support chats.

The biggest trap for automation beginners is trying to locate an element inside an iFrame using `page.locator()`. 

**This will fail.** The parent page (`page`) has no visibility into the child iFrame's DOM.

### The Selenium Way vs. The Playwright Way

In older tools, interacting with an iFrame was a tedious multi-step process. You had to explicitly command the driver to `switchTo().frame()`, perform your actions, and then explicitly `switchTo().defaultContent()` to return to the main page.

Playwright makes this completely obsolete using **`frameLocator()`**. 

With a `frameLocator`, you don't actually "switch" your context. You simply create a tunnel directly to the iframe, and chain your standard locators onto it!

### Penetrating the iFrame

Let's test this against our live practice site (`https://practice.mycodeyatra.com/#/frames`):

Create `tests/blog17_iframes.spec.ts`:

```typescript
import { test, expect } from '@playwright/test';
test('Interacting with elements inside an iFrame', async ({ page }) => {
  await page.goto('https://practice.mycodeyatra.com/#/frames');
  
  // 1. If we try to find the button directly, Playwright will fail!
  // const btn = page.locator('#iframe-btn'); // THIS WILL FAIL
  
  // 2. Instead, we must first locate the iFrame itself using frameLocator()
  const frame = page.frameLocator('[data-testid="iframe-container"]');
  
  // 3. Now we can chain standard locators off of the frame!
  const frameButton = frame.locator('#iframe-btn');
  const messageText = frame.locator('#msg');
  
  // 4. Interact with the button INSIDE the frame
  await frameButton.click();
  
  // 5. Verify the text updated INSIDE the frame
  await expect(messageText).toHaveText('IFrame Button Clicked!');
  
  console.log('Successfully penetrated the iFrame and clicked the button!');
  
  // Note: If you want to click something on the main page now, 
  // you just use page.locator() like normal. No context switching required!
});
```

### Execution Output

When you run `npx playwright test tests/blog17_iframes.spec.ts`:

```
Running 1 test using 1 worker

Successfully penetrated the iFrame and clicked the button!
  OK   1 tests/blog17_iframes.spec.ts:4:7 > Blog 17: Handling Inline Frames (iFrames) > Interacting with elements inside an iFrame (712ms)

  1 passed (1.9s)
```

### Conclusion

Playwright's `frameLocator` is a massive quality-of-life improvement. By tunneling into the iframe instead of completely switching contexts, your scripts remain clean, linear, and completely immune to context-switching bugs.

In **Blog 18**, we will conquer one of the most mysterious parts of modern web architecture: **Shadow DOM Components**!
