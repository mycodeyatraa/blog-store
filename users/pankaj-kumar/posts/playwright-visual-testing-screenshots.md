---
title: "Screenshot Comparisons (Visual Regression Testing)"
date: "2025-04-02"
description: "Learn how to perform visual regression testing in Playwright TypeScript, capturing element-level screenshots and managing reference baselines."
tags: ["Playwright", "TypeScript", "Visual Testing", "Regression", "toHaveScreenshot"]
---

Welcome to Blog 50 of the **Playwright TypeScript Mastery Series**!

Functional assertions verify values, state, and element existence in the DOM. However, standard assertions cannot check if a button is partially obscured, a paragraph overflows its container, or a style sheet failed to load. To ensure the interface renders correctly, you need **Visual Regression Testing**.

Today, we will learn how to capture element screenshots, compare them against reference "golden" images, and manage baseline updates using Playwright.

---

### How Playwright Handles Visual Testing

Playwright has built-in visual comparison matching. When you call `expect(locator).toHaveScreenshot(name)`:
1. **First Run (No baseline)**: Playwright captures the actual image and writes it to a snapshots directory matching your test filename. The test fails, indicating a missing reference.
2. **Subsequent Runs**: Playwright captures the actual screenshot, compares it pixel-by-pixel against the reference baseline, and fails if the difference exceeds the default threshold.
3. **Updating Baselines**: When you introduce intentional UI changes, run the test runner with the `--update-snapshots` flag to overwrite older baselines.

---

### Step 1: Implementing the Visual Regression Spec

Create a test file `tests/blog50_visual_regression.spec.ts` demonstrating element-level visual regression:

```typescript
import { test, expect } from '@playwright/test';

test.describe('Blog 50: Visual Regression Testing in Playwright', () => {

  test('Capture and compare element screenshot', async ({ page }) => {
    // Navigate to a static page
    await page.goto('data:text/html,<html><body><div id="static-box" style="width: 100px; height: 100px; background-color: blue;"></div></body></html>');
    
    // Locate the element we want to visually assert
    const staticBox = page.locator('#static-box');
    
    // Assert visual layout. On the first run, Playwright will save this as the reference image.
    await expect(staticBox).toHaveScreenshot('static-box.png');
    console.log('[Visual Regression] Visual verification completed.');
  });

});
```

---

### Step 2: Write Reference Baseline and Execute

On the first run, update or write the baseline image:

```
npx playwright test tests/blog50_visual_regression.spec.ts --update-snapshots
```

**Output:**

```
Running 1 test using 1 worker

A snapshot doesn't exist at D:\MyCodeYatra\AILearning2026\Repository\mcyt-plw-typescript\tests\blog50_visual_regression.spec.ts-snapshots\static-box-win32.png, writing actual.
[Visual Regression] Visual verification completed.
  ✓  1 tests/blog50_visual_regression.spec.ts:5:7 › Blog 50: Visual Regression Testing in Playwright › Capture and compare element screenshot (509ms)

  1 passed (1.8s)
```

Now run the test again to verify comparisons:

```
npx playwright test tests/blog50_visual_regression.spec.ts
```

**Output:**

```
Running 1 test using 1 worker

[Visual Regression] Visual verification completed.
  ✓  1 tests/blog50_visual_regression.spec.ts:5:7 › Blog 50: Visual Regression Testing in Playwright › Capture and compare element screenshot (318ms)

  1 passed (1.6s)
```

In the next blog, we will refine this by learning how to configure **Pixel-Perfect Assertions**, adjust pixel matching thresholds, and ignore dynamic elements during comparisons!
