---
title: "Executing Custom JavaScript"
date: "2025-04-05"
description: "Learn how to execute custom JavaScript scripts inside the browser context using page.evaluate in Playwright TypeScript."
tags: ["Playwright", "TypeScript", "JavaScript", "page.evaluate"]
---

Welcome to Blog 53 of the **Playwright TypeScript Mastery Series**!

While Playwright provides powerful locating and action APIs, sometimes you need to escape the testing library and interact with the page's JavaScript context directly. You might want to access window-level global states, calculate positions via scroll dimensions, retrieve details not exposed in the DOM attributes, or trigger manual events.

Today, we will learn how to run custom JS code in the browser using `page.evaluate()`.

---

### Understanding the Browser Context Execution

`page.evaluate()` runs a JavaScript function within the page context:
- It returns the result of the function execution back to the Node.js/test script.
- Arguments can be passed from the test script into the browser context. Passed arguments must be JSON-serializable.

---

### Step 1: Implementing the Custom JS Spec

Create a test file `tests/blog53_evaluate.spec.ts`:

```typescript
import { test, expect } from '@playwright/test';

test.describe('Blog 53: Executing Custom JavaScript with page.evaluate', () => {

  test('Evaluate simple expressions and return values', async ({ page }) => {
    await page.goto('data:text/html,<html><body><h1 id="title">Hello World</h1></body></html>');

    // 1. Get window properties
    const userAgent = await page.evaluate(() => navigator.userAgent);
    console.log(`[JS Evaluate] User Agent: ${userAgent}`);
    expect(userAgent).toBeTruthy();

    // 2. Query DOM attributes directly
    const titleText = await page.evaluate(() => {
      const heading = document.getElementById('title');
      return heading ? heading.innerText : '';
    });
    expect(titleText).toBe('Hello World');
  });

  test('Pass arguments to page.evaluate', async ({ page }) => {
    await page.goto('data:text/html,<html><body><div id="target">Original</div></body></html>');

    const newText = 'Playwright Evaluated Text';
    
    // Pass local variables into browser window evaluation context
    await page.evaluate((textToSet) => {
      const el = document.getElementById('target');
      if (el) el.innerText = textToSet;
    }, newText);

    const actualText = await page.locator('#target').innerText();
    expect(actualText).toBe(newText);
  });

});
```

---

### Step 2: Running the Test

Execute the test via Playwright CLI:

```
npx playwright test tests/blog53_evaluate.spec.ts
```

**Output:**

```
Running 2 tests using 1 worker

[JS Evaluate] User Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) HeadlessChrome/149.0.7827.55 Safari/537.36
  ✓  1 tests/blog53_evaluate.spec.ts:5:7 › Blog 53: Executing Custom JavaScript with page.evaluate › Evaluate simple expressions and return values (224ms)
  ✓  2 tests/blog53_evaluate.spec.ts:21:7 › Blog 53: Executing Custom JavaScript with page.evaluate › Pass arguments to page.evaluate (174ms)

  2 passed (1.8s)
```

In the next blog, we will build upon custom JS executions to learn how to **manipulate LocalStorage and SessionStorage** inside Playwright!
