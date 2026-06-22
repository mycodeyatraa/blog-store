---
title: "First E2E Test with Playwright Test"
date: "2025-02-20"
description: "It's time to build a real test! We will combine our knowledge of user-centric locators and auto-retrying assertions to build a complete End-to-End Login scenario."
tags: ["Playwright", "TypeScript", "E2E", "Testing", "Login"]
---

Welcome to Blog 8 of the **Playwright TypeScript Mastery Series**!

So far, we have covered the theoretical foundations of Playwright. We know how the BrowserContext works, how to use user-centric locators like `getByRole`, and how to validate state using `expect()`.

Now, it is time to put all the pieces together. Today, we will write our first complete, production-ready End-to-End (E2E) Test Suite for a Login Page!

### The Scenarios

We are going to write two distinct scenarios for our application (`https://practice.mycodeyatra.com/#/login`):
1.  **Positive Case**: A user enters valid credentials, clicks Login, and is successfully routed to the Dashboard.
2.  **Negative Case**: A user enters an invalid password, clicks Login, and an error message is displayed.

### Writing the Test Suite

Create a new file in your project: `tests/blog8_e2e_login.spec.ts`.

```typescript
import { test, expect } from '@playwright/test';

test.describe('Blog 8: First E2E Test', () => {
  // ==========================================
  // SCENARIO 1: Successful Login
  // ==========================================
  test('Successful Login Scenario', async ({ page }) => {
    // 1. Navigate to the application
    console.log('Navigating to the login page...');
    await page.goto('https://practice.mycodeyatra.com/#/login');
    // 2. Locate the Username field and Type
    console.log('Entering username...');
    await page.getByPlaceholder('Enter Username').fill('admin');
    // 3. Locate the Password field and Type
    console.log('Entering password...');
    await page.getByPlaceholder('Enter Password').fill('password123');
    // 4. Locate the Login Button and Click
    console.log('Clicking the Login button...');
    await page.getByRole('button', { name: 'Login' }).click();
    // 5. Assert that the login was successful by checking the URL
    console.log('Validating successful login...');
    await expect(page).toHaveURL(/.*dashboard/);
    // 6. Assert that a success banner appears
    const successAlert = page.locator('#success-alert');
    await expect(successAlert).toBeVisible();
    await expect(successAlert).toHaveText(/Welcome back/);
  });
  // ==========================================
  // SCENARIO 2: Failed Login
  // ==========================================
  test('Failed Login Scenario (Invalid Password)', async ({ page }) => {
    await page.goto('https://practice.mycodeyatra.com/#/login');
    // Perform the invalid login
    await page.getByPlaceholder('Enter Username').fill('admin');
    await page.getByPlaceholder('Enter Password').fill('wrongpassword');
    await page.getByRole('button', { name: 'Login' }).click();
    // Assert that an error message is displayed
    const errorAlert = page.locator('#error-alert');
    await expect(errorAlert).toBeVisible();
    await expect(errorAlert).toHaveText('Invalid credentials');
    // Use the .not modifier to assert the URL did NOT change!
    await expect(page).not.toHaveURL(/.*dashboard/);
  });
});
```

### Breaking Down the Code

*   **`page.goto()`**: This is a core Playwright function. It tells the browser to navigate to a URL and wait until the page has fully loaded before executing the next line of code.
*   **`.fill()`**: In older frameworks like Selenium, you usually use `.sendKeys()`. Playwright's `.fill()` is vastly superior because it automatically clears the input box before typing the new text!
*   **`.click()`**: This command natively waits for the button to be visible, enabled, and perfectly stable before triggering the click event.
*   **`.toHaveURL(/.*dashboard/)`**: Notice the `/ /` syntax? That is a Regular Expression! We don't care what the exact IP address or domain is; as long as the test ends on "dashboard", it passes!
*   **`expect(page).not`**: By appending `.not`, we can assert negative conditions (e.g., verifying that a user is *blocked* from accessing the dashboard).

### Execution Output

Open your terminal and run the test in headed mode:

```bash
npx playwright test tests/blog8_e2e_login.spec.ts --headed
```

**Output:**

```text
Running 2 tests using 1 worker
Navigating to the login page...
Entering username...
Entering password...
Clicking the Login button...
Validating successful login...
  ✓  1 Successful Login Scenario (1.8s)
  ✓  2 Failed Login Scenario (Invalid Password) (1.4s)
  2 passed (3.2s)
```

### Conclusion

Congratulations! You have written your first professional, production-ready End-to-End test suite using Playwright and TypeScript. You navigated, interacted, and asserted seamlessly.

However, writing tests is only half the battle. What happens when a test fails? In **Blog 9**, we will explore Playwright's incredible **Debugging Tools**, including the Inspector and the revolutionary Trace Viewer!
