---
title: "Handling Multiple Elements"
date: "2025-02-23"
description: "What happens when a locator returns 50 elements instead of just 1? Learn how to iterate through elements and select specific items using first(), last(), nth(), and all()."
tags: ["Playwright", "TypeScript", "Locators", "Multiple Elements", "nth"]
---

Welcome to Blog 11 of the **Playwright TypeScript Mastery Series**!

One of the most common errors beginners face in Playwright is the dreaded **"strict mode violation"**. 

By default, Playwright operates in strict mode. If you write `await page.locator('.product-card').click()`, but there are 10 product cards on the screen, Playwright will throw an error and crash! It refuses to randomly guess which of the 10 elements you intended to click.

Today, we will learn how to resolve this by specifically handling multiple elements.

### Selecting by Index

If your locator returns multiple elements, you can use built-in indexing methods to specify exactly which one you want.

Create a file `tests/blog11_multiple_elements.spec.ts`:

```typescript
import { test, expect } from '@playwright/test';
test('Selecting specific elements from a list', async ({ page }) => {
  await page.goto('https://practice.mycodeyatra.com/#/products');
  // This locator returns 10 elements!
  const productCards = page.locator('.product-card');
  // 1. Select the VERY FIRST element
  const firstProduct = productCards.first();
  await firstProduct.click();
  // 2. Select the VERY LAST element
  const lastProduct = productCards.last();
  await expect(lastProduct).toBeVisible();
  // 3. Select the 3rd element (Remember: arrays are Zero-indexed!)
  const thirdProduct = productCards.nth(2);
  await expect(thirdProduct).toBeVisible();
});
```

Using `.first()`, `.last()`, and `.nth(index)` allows you to completely bypass the strict mode violation because you are explicitly telling Playwright which specific element out of the 10 to interact with.

### Iterating Over Elements

What if you don't want to just click one element? What if you want to extract the text from *all 10* product cards and print them to the console?

To do this, we must convert the Playwright `Locator` object into a standard TypeScript `Array`. We do this using the `.all()` method!

```typescript
test('Iterating over all elements', async ({ page }) => {
  await page.goto('https://practice.mycodeyatra.com/#/products');
  const productCards = page.locator('.product-card');
  // .all() resolves the locator and returns an Array of Locators!
  const allProducts = await productCards.all();
  console.log(`Found ${allProducts.length} products on the page!`);
  // Use a standard JavaScript 'for...of' loop to iterate
  for (const product of allProducts) {
    const textContent = await product.textContent();
    console.log(`Product Name: ${textContent}`);
  }
});
```

### Important Warning about `.all()`

You must be very careful when using `.all()`. 

Standard Playwright matchers (like `.click()`) are **auto-retrying**. If the element isn't on the screen yet, they will wait for it. 

However, `.all()` is **NOT auto-retrying**. It executes immediately. If the web page hasn't finished loading yet, `.all()` might return an array of `0` elements, and your test will silently skip the loop! 

If you are dealing with a slow-loading list, it is best practice to wait for the first element to appear *before* calling `.all()`:

```typescript
// Wait for at least one product to appear on the screen!
await productCards.first().waitFor();
// Now it is safe to grab all of them
const allProducts = await productCards.all();
```

### Execution Output

When you run `npx playwright test tests/blog11_multiple_elements.spec.ts`:

```text
Running 2 tests using 1 worker
  ✓  1 Selecting specific elements from a list (1.2s)
Found 10 products on the page!
Product Name: Wireless Mouse
Product Name: Mechanical Keyboard
Product Name: 4K Monitor
...
  ✓  2 Iterating over all elements (1.8s)
  2 passed (3.0s)
```

### Conclusion

You now know how to handle strict mode violations gracefully using `.first()`, `.last()`, and `.nth()`, and how to iterate through large tables and lists using `.all()`.

In **Blog 12**, we are moving to a brand new topic. It is time to master **Form Interactions**, starting with Dropdowns and Select Menus!
