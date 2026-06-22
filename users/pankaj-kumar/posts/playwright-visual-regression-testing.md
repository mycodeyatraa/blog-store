---
title: "Visual Regression Testing"
date: "2025-03-12"
description: "Go beyond checking text. Learn how to catch visual bugs (like broken CSS or misaligned elements) using Playwright's built-in image comparison features."
tags: ["Playwright", "TypeScript", "Visual Testing", "Screenshots"]
---

Welcome to Blog 28 of the **Playwright TypeScript Mastery Series**!

So far in this series, our tests have been purely functional. We check if an element exists, if the text is correct, or if a button is clickable. 

But what if a developer accidentally deletes a CSS file? The button might technically still exist and be clickable, but it might be completely invisible to a real human, or the entire layout might be misaligned!

Traditional functional tests cannot catch these issues. We need **Visual Regression Testing**.

### How Playwright Visual Testing Works

Playwright has visual testing built directly into the framework via the `toHaveScreenshot()` assertion.

When you run a visual test for the **first time**, Playwright will take a screenshot of the element (or the entire page) and save it in a folder called `tests/your-spec-name-snapshots`. This is known as the **Golden Baseline**. The test will intentionally "fail" this first time because it had nothing to compare against.

When you run the test the **second time** (and every time thereafter), Playwright takes a new screenshot and compares it pixel-by-pixel to the Golden Baseline. If the pixels don't match, the test fails!

### Writing a Visual Regression Test

Let's write a script to take visual snapshots of our practice site.

Create `tests/blog28_visual_testing.spec.ts`:

```typescript
import { test, expect } from '@playwright/test';
 
test.describe('Blog 28: Visual Regression Testing', () => {
 
  test('Verify Home Page Layout using Screenshots', async ({ page }) => {
    // Navigate to the practice site
    await page.goto('https://practice.mycodeyatra.com');
    
    // Wait for the page to be fully loaded and stable
    await expect(page.locator('h1').first()).toBeVisible();
 
    // 1. Take a screenshot of the entire page and compare it to the baseline!
    // NOTE: On the very first run, this test will "fail" because there is no baseline.
    // Playwright will automatically save the first screenshot as the golden baseline.
    // Future runs will compare against that baseline!
    await expect(page).toHaveScreenshot('home-page-layout.png', {
      maxDiffPixels: 100 // Allow a tiny bit of anti-aliasing difference
    });
 
    // 2. We can also take a screenshot of a specific element instead of the whole page!
    const headerTitle = page.locator('h1').first();
    await expect(headerTitle).toHaveScreenshot('header-title.png');
    
    console.log('Visual Regression completed!');
  });
 
});
```

### Execution Output (Second Run)

After running the test once to generate the baselines, run it again:

`npx playwright test tests/blog28_visual_testing.spec.ts`

```
Running 1 test using 1 worker
 
Visual Regression completed!
  OK  1 tests/blog28_visual_testing.spec.ts:5:7 > Blog 28: Visual Regression Testing > Verify Home Page Layout using Screenshots (1.5s)
 
  1 passed (3.7s)
```

### Updating Baselines

If your application legitimately changes (e.g., you updated the logo or changed the brand colors), your visual tests will correctly fail. 

To tell Playwright that the new visual state is the *new* correct state, you simply run your tests with the `--update-snapshots` flag:

`npx playwright test --update-snapshots`

This will overwrite all the old Golden Baselines with the new, correct screenshots.

### Conclusion

Visual Regression Testing is an incredibly powerful safety net that catches CSS regressions and layout shifts that traditional functional tests miss entirely.

In **Blog 29**, we will explore how to interact with the Operating System by **Uploading and Downloading Files**!
