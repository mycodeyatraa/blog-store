---
title: "Locators Chaining and Filtering"
date: "2025-02-22"
description: "Web pages are complex. When simple locators aren't enough, learn how to chain locators together and use advanced filtering techniques (like hasText) to find nested elements."
tags: ["Playwright", "TypeScript", "Locators", "Chaining", "Filtering"]
---

Welcome to Blog 10 of the **Playwright TypeScript Mastery Series**!

In Blog 6, we learned how to use user-centric locators like `page.getByRole()`. That works perfectly for simple web pages. 

But what happens when you are testing a complex enterprise application? What if there are **three** different "Submit" buttons on the exact same page, and you need to click the one inside the second modal window? 

Today, we will master **Chaining** and **Filtering** to conquer complex DOM structures.

### 1. Chaining Locators

In older frameworks, finding nested elements often required writing massive, unreadable XPath strings like `//div[@class='modal']//form//button[text()='Submit']`.

Playwright offers a much cleaner approach: **Locator Chaining**.

You can append locators together to narrow down your search, moving from parent to child.

Create a file `tests/blog10_locators_chaining.spec.ts`:

```typescript
import { test, expect } from '@playwright/test';
 
test('Chaining Locators', async ({ page }) => {
  await page.goto('https://practice.mycodeyatra.com/#/login');
  
  // 1. Find the parent container first
  const loginForm = page.locator('.login-form-container');
  
  // 2. Chain off the parent to find the specific child
  const loginButton = loginForm.getByRole('button', { name: 'Login' });
  
  await expect(loginButton).toBeVisible();
});
```

Because `loginButton` is chained to `loginForm`, Playwright will *only* search for a Login button that exists physically inside that specific container.

### 2. Filtering Locators

Chaining is great for Parent-Child relationships. But what if you are dealing with a massive list or a data table where all the elements look identical?

Imagine a table with 50 rows of users. You need to click the "Edit" button for the user named "John Doe". 

You can use the `.filter()` method!

```typescript
test('Filtering Locators with hasText', async ({ page }) => {
  await page.goto('https://practice.mycodeyatra.com/#/dashboard');
  
  // 1. Get ALL rows in the table (returns an array of 50 locators)
  const allRows = page.locator('table tr');
  
  // 2. Filter down to ONLY the row containing the text 'John Doe'
  const johnDoeRow = allRows.filter({ hasText: 'John Doe' });
  
  // 3. Chain off that specific row to click its specific Edit button!
  const editButton = johnDoeRow.getByRole('button', { name: 'Edit' });
  
  await editButton.click();
});
```

### Advanced Filtering Strategies

You are not limited to just `hasText`. Playwright offers several filtering options:

*   **`hasText`**: Filters elements containing specific text.
*   **`hasNotText`**: Filters out elements containing specific text.
*   **`has`**: Filters elements that *contain another specific locator*.

Example of `has`:

```typescript
// Find a row that physically contains an SVG Checkmark icon
const completedRow = allRows.filter({ has: page.locator('svg.checkmark-icon') });
```

### Execution Output

When you run `npx playwright test tests/blog10_locators_chaining.spec.ts`:

```
Running 2 tests using 1 worker
  OK   1 Chaining Locators (1.1s)
Successfully found the Edit button for John Doe!
  OK   2 Filtering Locators with hasText (1.4s)
 
  2 passed (2.5s)
```

### Conclusion

You no longer need to write brittle, mile-long XPath queries. By combining **Chaining** and **Filtering**, you can break down the most complex web applications into simple, readable, and highly stable TypeScript code!

In **Blog 11**, we will cover the final piece of the Locator puzzle: **Handling Multiple Elements** and iterating through lists!
