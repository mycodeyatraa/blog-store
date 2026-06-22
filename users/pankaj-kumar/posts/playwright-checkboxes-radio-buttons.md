---
title: "Checkboxes and Radio Buttons"
date: "2025-02-25"
description: "Learn how to robustly interact with checkboxes and radio buttons in Playwright using .check() and .uncheck(), and how to assert their status."
tags: ["Playwright", "TypeScript", "Checkbox", "Radio", "Forms"]
---

Welcome to Blog 13 of the **Playwright TypeScript Mastery Series**!

We continue our Deep Dive into Form Interactions. In the previous blog, we mastered dropdowns. Today, we will conquer two of the most common form elements on the web: **Checkboxes** and **Radio Buttons**.

### The Problem with `.click()`

Beginners often try to select a checkbox by using `await page.locator('#my-checkbox').click()`. 

While this technically works, it is dangerous! What if the checkbox was *already* checked because of a default user preference? Using `.click()` would uncheck it, ruining your test!

To solve this, Playwright provides the `.check()` and `.uncheck()` methods.

### 1. Interacting with Checkboxes

The `.check()` method is smart. If the checkbox is already checked, `.check()` does absolutely nothing. It guarantees the final state of the element is checked.

Let's test this on our live practice site (`https://practice.mycodeyatra.com/#/form-practice`):

Create a file `tests/blog13_checkboxes.spec.ts`:
```typescript
import { test, expect } from '@playwright/test';
test('Checking and Unchecking Checkboxes', async ({ page }) => {
  await page.goto('https://practice.mycodeyatra.com/#/form-practice');
  // Locate the "Automation" and "DevOps" interest checkboxes
  const automationCheckbox = page.getByTestId('interest-automation');
  const devopsCheckbox = page.getByTestId('interest-devops');
  // Use .check() to select them
  await automationCheckbox.check();
  await devopsCheckbox.check();
  // Verify they are checked
  await expect(automationCheckbox).toBeChecked();
  await expect(devopsCheckbox).toBeChecked();
  // Use .uncheck() to deselect
  await devopsCheckbox.uncheck();
  await expect(devopsCheckbox).not.toBeChecked();
});
```

### 2. Interacting with Radio Buttons

Radio buttons work identically to checkboxes, with one logical difference: you cannot `.uncheck()` a radio button. Selecting one radio button automatically deselects the others in the same group.
```typescript
test('Selecting Radio Buttons', async ({ page }) => {
  await page.goto('https://practice.mycodeyatra.com/#/form-practice');
  // Locate the Gender radio buttons
  const maleRadio = page.getByTestId('gender-male');
  const femaleRadio = page.getByTestId('gender-female');
  // Check the Female radio button
  await femaleRadio.check();
  // Assert Female is checked and Male is NOT checked
  await expect(femaleRadio).toBeChecked();
  await expect(maleRadio).not.toBeChecked();
});
```

### Execution Output

When you run `npx playwright test tests/blog13_checkboxes.spec.ts`:
```bash
Running 2 tests using 1 worker

  ✓  1 tests\blog13_checkboxes.spec.ts:4:7 › Blog 13: Checkboxes and Radio Buttons › Checking and Unchecking Checkboxes (854ms)
  ✓  2 tests\blog13_checkboxes.spec.ts:24:7 › Blog 13: Checkboxes and Radio Buttons › Selecting Radio Buttons (632ms)

  2 passed (3.1s)
```

### Conclusion

Always use `.check()` and `.uncheck()` instead of `.click()` when dealing with checkboxes and radio buttons. It ensures your tests remain stable regardless of the default state of the web application.

In **Blog 14**, we will cover the final piece of Form Interaction: **Handling Alerts, Prompts, and Confirmation Dialogs**!
