---
title: "Handling Multiple Elements"
date: "2025-02-23"
description: "What happens when a locator returns 50 elements instead of just 1? Learn how to iterate through elements and select specific items using first(), last(), nth(), and all()."
tags: ["Playwright", "TypeScript", "Locators", "Multiple Elements", "nth"]
---

Welcome to Blog 11 of the **Playwright TypeScript Mastery Series**!

*(Note: For this tutorial, we will be using our live practice environment at **`https://practice.mycodeyatra.com/#/sandbox`** so you can follow along with the code exactly!)*

One of the most common errors beginners face in Playwright is the dreaded **"strict mode violation"**. 

By default, Playwright operates in strict mode. If you write `await page.locator('.hover-card').click()`, but there are 11 tiles on the screen matching that class, Playwright will throw an error and crash! It refuses to randomly guess which of the elements you intended to click.

Today, we will learn how to resolve this by specifically handling multiple elements.

### Selecting by Index

If your locator returns multiple elements, you can use built-in indexing methods to specify exactly which one you want.

Create a file `tests/blog11_multiple_elements.spec.ts`:

```typescript
import { test, expect } from '@playwright/test';
test('Selecting specific elements from a list', async ({ page }) => {
  // Navigating to the live Sandbox!
  await page.goto('https://practice.mycodeyatra.com/#/sandbox');
  // This locator returns multiple elements!
  const sandboxTiles = page.locator('.hover-card');
  // 1. Select the VERY FIRST element
  const firstTile = sandboxTiles.first();
  await expect(firstTile).toBeVisible();
  // 2. Select the VERY LAST element
  const lastTile = sandboxTiles.last();
  await expect(lastTile).toBeVisible();
  // 3. Select the 3rd element (Remember: arrays are Zero-indexed!)
  const thirdTile = sandboxTiles.nth(2);
  await expect(thirdTile).toBeVisible();
});
```

Using `.first()`, `.last()`, and `.nth(index)` allows you to completely bypass the strict mode violation because you are explicitly telling Playwright which specific element out of the list to interact with.

### Iterating Over Elements

What if you don't want to just click one element? What if you want to extract the text from *all 11* tiles and print them to the console?

To do this, we must convert the Playwright `Locator` object into a standard TypeScript `Array`. We do this using the `.all()` method!

```typescript
test('Iterating over all elements', async ({ page }) => {
  await page.goto('https://practice.mycodeyatra.com/#/sandbox');
  const sandboxTiles = page.locator('.hover-card');
  // .all() resolves the locator and returns an Array of Locators!
  const allTiles = await sandboxTiles.all();
  console.log(`Found ${allTiles.length} tiles on the page!`);
  // Use a standard JavaScript 'for...of' loop to iterate
  for (const tile of allTiles) {
    const titleElement = tile.locator('h3');
    const textContent = await titleElement.textContent();
    console.log(`Tile Title: ${textContent}`);
  }
});
```

### Execution Output

When you run `npx playwright test tests/blog11_multiple_elements.spec.ts`:

```text
Running 2 tests using 1 worker

  OK   1 tests/blog11_multiple_elements.spec.ts:4:7 > Blog 11: Handling Multiple Elements > Selecting specific elements from a list (3.8s)
Found 11 tiles on the page!
Tile Title: Form Practice
Tile Title: Authentication & Session
Tile Title: Dropdown Lists
Tile Title: Alerts & Overlays
Tile Title: Dynamic Content
Tile Title: Data Tables & Grids
Tile Title: Mouse Actions
Tile Title: Shadow DOM Components
Tile Title: Frames & iFrames
Tile Title: Complex Widgets
Tile Title: File Upload & Download
  OK   2 tests/blog11_multiple_elements.spec.ts:24:7 > Blog 11: Handling Multiple Elements > Iterating over all elements (620ms)

  2 passed (6.1s)
```

### Conclusion

You now know how to handle strict mode violations gracefully using `.first()`, `.last()`, and `.nth()`, and how to iterate through large tables and lists using `.all()`.

In **Blog 12**, we are moving to a brand new topic. It is time to master **Form Interactions**, starting with Dropdowns and Select Menus!
