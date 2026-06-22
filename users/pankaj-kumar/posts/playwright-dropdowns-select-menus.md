---
title: "Dropdowns and Select Menus"
date: "2025-02-24"
description: "Master form interactions! Learn how to seamlessly select options from both native <select> menus and modern custom dropdowns using Playwright."
tags: ["Playwright", "TypeScript", "Dropdown", "Select", "Forms"]
---

Welcome to Blog 12 of the **Playwright TypeScript Mastery Series**!

Now that we know how to identify elements and iterate through lists, we are going to spend the next few blogs mastering **Form Interactions**.

Forms are the backbone of any web application. The most notoriously difficult form element for beginners to automate is the Dropdown menu. 

Today, we will learn how to interact with the two completely different types of dropdowns: **Native `<select>` Menus** and **Custom UI Dropdowns**.

### 1. Native `<select>` Menus

A native dropdown uses the standard HTML `<select>` and `<option>` tags. These are the easiest to automate because Playwright has a dedicated built-in method for them: `.selectOption()`.

Let's write a test using the live sandbox (`https://practice.mycodeyatra.com/#/sandbox`):

Create a file `tests/blog12_dropdowns.spec.ts`:

```typescript
import { test, expect } from '@playwright/test';
test('Interacting with a Native Select Menu', async ({ page }) => {
  await page.goto('https://practice.mycodeyatra.com/#/sandbox');
  // Locate the native <select> element
  const nativeDropdown = page.locator('select#country-dropdown');
  // 1. Select by the underlying VALUE attribute
  await nativeDropdown.selectOption('IN');
  // 2. Select by the VISIBLE TEXT shown to the user
  await nativeDropdown.selectOption({ label: 'United States' });
  // 3. Select by INDEX (Selects the 3rd option in the list)
  await nativeDropdown.selectOption({ index: 2 });
  // Assert that the dropdown successfully updated its value
  await expect(nativeDropdown).toHaveValue('US');
});
```

*Note: You do **not** need to click the dropdown first. Playwright's `.selectOption()` automatically handles everything behind the scenes!*

### 2. Custom UI Dropdowns (Non-Select)

Modern web frameworks like React and Angular rarely use native `<select>` tags because they are difficult to style with CSS. Instead, developers build "Fake Dropdowns" using `<div>`, `<ul>`, and `<li>` tags styled to look like a menu.

Because these are not real `<select>` tags, you **cannot** use `.selectOption()`. If you try, Playwright will throw an error!

To automate a Custom Dropdown, you must manually simulate human behavior:

```typescript
test('Interacting with a Custom Dropdown (Non-Select)', async ({ page }) => {
  await page.goto('https://practice.mycodeyatra.com/#/sandbox');
  // 1. You MUST physically click the dropdown box to expand the menu first!
  await page.locator('div.custom-dropdown-header').click();
  // 2. Locate the specific option from the expanded list and click it
  const optionToSelect = page.getByRole('listitem', { name: 'Dark Mode' });
  await optionToSelect.click();
  // 3. Assert the UI updated correctly
  const selectedText = page.locator('div.custom-dropdown-header');
  await expect(selectedText).toHaveText('Dark Mode');
});
```

### Execution Output

When you run `npx playwright test tests/blog12_dropdowns.spec.ts`:

```text
Running 2 tests using 1 worker
  ✓  1 Interacting with a Native Select Menu (1.1s)
  ✓  2 Interacting with a Custom Dropdown (Non-Select) (1.4s)
  2 passed (2.5s)
```

### Conclusion

Handling dropdowns is simple once you identify the underlying architecture. Always right-click the dropdown in your browser and select **Inspect**.
*   If it is a `<select>` tag, use Playwright's awesome `.selectOption()` method.
*   If it is a `<div>` or `<ul>`, you must physically `.click()` the box and then `.click()` the option.

In **Blog 13**, we will tackle another tricky form element: **Checkboxes and Radio Buttons**!
