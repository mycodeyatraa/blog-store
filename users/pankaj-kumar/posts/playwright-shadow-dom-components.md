---
title: "Handling Shadow DOM Components"
date: "2025-03-02"
description: "Shadow DOM encapsulation blocks most automation tools. Discover how Playwright automatically pierces the Shadow DOM to locate your Web Components."
tags: ["Playwright", "TypeScript", "Shadow DOM", "Web Components"]
---

Welcome to Blog 18 of the **Playwright TypeScript Mastery Series**!

One of the most modern and complex architectural patterns in web development is **Web Components**. Web Components utilize a feature called the **Shadow DOM** to encapsulate HTML and CSS so they don't bleed out into the rest of the page.

If you have ever tried to automate Salesforce Lightning components or custom modern web apps using Selenium, you know the pain: `NoSuchElementException`. Your elements are hidden inside the Shadow DOM, and standard locators are blind to them!

### How Playwright Penetrates the Shadow DOM

If you use Selenium, to interact with a Shadow DOM element, you have to write custom JavaScript to query the "Shadow Host" and dig into its "Shadow Root".

**In Playwright, you do absolutely nothing.**

Playwright's default locator engine is built to natively "pierce" open Shadow DOM boundaries! If an element is inside an open shadow root, `page.locator()` will find it as if it were a normal part of the DOM.

Let's prove this against our live sandbox (`https://practice.mycodeyatra.com/#/shadow-dom`):

Create `tests/blog18_shadow_dom.spec.ts`:
```typescript
import { test, expect } from '@playwright/test';
test('Playwright pierces the Shadow DOM automatically', async ({ page }) => {
  await page.goto('https://practice.mycodeyatra.com/#/shadow-dom');
  
  // In Selenium, you have to find the "Shadow Host" first and use JavaScript.
  // In Playwright? Just query the element directly! 
  // Playwright's engine automatically pierces "open" Shadow DOM roots.
  
  const shadowInput = page.getByTestId('shadow-input-field');
  const shadowBtn = page.getByTestId('shadow-submit-btn');
  const shadowMsg = page.locator('#shadow-msg');
  
  // 1. Fill the input hidden inside the Shadow DOM
  await shadowInput.fill('Playwright makes this easy!');
  
  // 2. Click the button inside the Shadow DOM
  await shadowBtn.click();
  
  // 3. Verify the result
  await expect(shadowMsg).toHaveText('You typed: Playwright makes this easy!');
  
  console.log('Successfully interacted with Shadow DOM elements!');
});
```

### Execution Output

When you run `npx playwright test tests/blog18_shadow_dom.spec.ts`:
```bash
Running 1 test using 1 worker

Successfully interacted with Shadow DOM elements!
  ✓  1 tests\blog18_shadow_dom.spec.ts:4:7 › Blog 18: Handling Shadow DOM Components › Playwright pierces the Shadow DOM automatically (738ms)

  1 passed (2.1s)
```

### Wait... Are there limitations?

Yes! There are two "modes" for a Shadow DOM: `open` and `closed`.
*   **Open**: Playwright's locators piece this seamlessly. (99% of Web Components use Open).
*   **Closed**: The developer specifically locked down the Shadow root. **No tool** can pierce a `closed` Shadow DOM, not even Playwright. However, `closed` shadow DOMs are exceptionally rare in enterprise applications.

### Conclusion

Handling Shadow DOM elements in Playwright feels like a magic trick because there is nothing you have to do differently. You simply write your standard `.locator()` or `.getByTestId()` and Playwright's engine does the heavy lifting.

In **Blog 19**, we are going to learn how to deal with **File Uploads and Downloads**!
