---
title: "Locators Deep Dive"
date: "2025-02-18"
description: "Stop relying on brittle CSS and XPath! Discover Playwright's modern approach to element identification using Role, Text, and user-centric locator strategies."
tags: ["Playwright", "TypeScript", "Locators", "CSS", "XPath"]
---

Welcome to Blog 6 of the **Playwright TypeScript Mastery Series**!

To interact with a web page, you first need to "find" the element you want to click or type into. In legacy frameworks, this was almost always done using CSS Selectors or XPath.

While Playwright fully supports CSS and XPath, it strongly encourages a paradigm shift: **User-Centric Locators**. Microsoft's philosophy is that you should locate elements exactly the same way a real human user (or an accessibility screen reader) locates them.

Today, we will write our first TypeScript test and explore these powerful locator strategies.

### 1. The Legacy Way: CSS and XPath

If you are transitioning from Selenium, you can still use your existing CSS and XPath knowledge. In Playwright, you use the generic `page.locator()` method.

Create a file at `tests/blog6_locators.spec.ts`:

```typescript
import { test, expect } from '@playwright/test';
 
test.describe('Blog 6: Locators Deep Dive', () => {
 
  test('Identify elements using CSS and XPath', async ({ page }) => {
    await page.goto('https://practice.mycodeyatra.com/#/login');
 
    console.log('Using CSS Selector...');
    const usernameInput = page.locator('input#username');
    await usernameInput.fill('admin');
 
    console.log('Using XPath...');
    const passwordInput = page.locator('//input[@id="password"]');
    await passwordInput.fill('password123');
  });
 
});
```

*   **`test.describe`**: Groups related tests together (similar to a Test Class).
*   **`test()`**: Defines an individual test case.
*   **`async ({ page }) => {}`**: This is Playwright injecting a fresh `Page` object directly into your test.

While CSS and XPath work, they are heavily tied to the DOM structure. If a developer changes `id="username"` to `class="user-input"`, your test breaks!

### 2. The Modern Way: User-Centric Locators

A real human user doesn't look at the screen and say, *"Let me click the element with id='submit-btn'."* They look at the screen and say, *"Let me click the button that says Login."*

Playwright provides built-in `getBy...` methods to mimic this exact behavior.

```typescript
  test('Identify elements using user-centric locators', async ({ page }) => {
    await page.goto('https://practice.mycodeyatra.com/#/login');
 
    console.log('Using getByRole (Recommended)...');
    // Finds a button specifically labeled "Login"
    const loginButton = page.getByRole('button', { name: 'Login' });
    
    console.log('Using getByText...');
    // Finds exact visible text on the page
    const formHeader = page.getByText('Login Portal');
 
    // Optional: Assert that they actually exist
    await expect(loginButton).toBeVisible();
    await expect(formHeader).toBeVisible();
  });
```

### Why `getByRole` is the Ultimate Locator

`getByRole` is the most powerful locator in Playwright. It locates elements by their implicit ARIA (Accessible Rich Internet Applications) role. 

Roles include: `button`, `checkbox`, `heading`, `link`, `textbox`, etc.

**Why use it?**
1.  **Resilience**: Developers frequently change CSS classes, but they rarely change the fact that a button is a `<button>`. Your tests become incredibly stable.
2.  **Accessibility (A11y)**: If your test can find the button using `getByRole`, it guarantees that visually impaired users utilizing Screen Readers can also find it! You are testing functionality *and* accessibility at the same time.

### Execution Output

When we run `npx playwright test tests/blog6_locators.spec.ts --headed`:

```
Running 2 tests using 1 worker
Using CSS Selector...
Using XPath...
  OK   1 Identify elements using CSS and XPath (1.2s)
Using getByRole (Recommended)...
Using getByText...
  OK   2 Identify elements using user-centric locators (0.9s)
 
  2 passed (2.1s)
```

### Conclusion

Whenever you write a new test in Playwright, your locator priority should always be:
1.  `page.getByRole()` (Best practice)
2.  `page.getByText()`
3.  `page.getByLabel()`
4.  `page.getByTestId()` (If your devs use `data-testid` attributes)
5.  `page.locator()` with CSS/XPath (Only as a last resort)

In **Blog 7**, we will explore the `expect()` function in depth and learn how to write Hard and Soft **Assertions** with Playwright Test!
