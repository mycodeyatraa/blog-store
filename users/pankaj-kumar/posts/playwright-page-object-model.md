---
title: "Implementing the Page Object Model (POM)"
date: "2025-03-08"
description: "Scale your test automation safely. Discover how the Page Object Model drastically reduces maintenance by separating locators from test logic."
tags: ["Playwright", "TypeScript", "POM", "Page Object Model", "Architecture"]
---

Welcome to Blog 25 of the **Playwright TypeScript Mastery Series**!

Imagine you have 50 different tests that all log into your application using `page.getByTestId('username')`. 

One day, the development team decides to change `data-testid="username"` to `data-testid="email"`. You now have to open 50 files and manually update the locator 50 times. 

This is the exact problem the **Page Object Model (POM)** solves.

### What is the Page Object Model?

POM is an architectural design pattern that separates **Locators and Actions** from **Test Logic**.

Instead of writing locators directly inside your test files, you create a dedicated Class file for each page in your application (e.g., `LoginPage.ts`, `DashboardPage.ts`). 

### Creating the Page Object

Let's model our live Login page (`https://practice.mycodeyatra.com/#/login`).

Create a new directory called `pages`, and inside it create `LoginPage.ts`:

```typescript
import { expect, Locator, Page } from '@playwright/test';
 
export class LoginPage {
  readonly page: Page;
  readonly usernameInput: Locator;
  readonly passwordInput: Locator;
  readonly loginButton: Locator;
  readonly headerText: Locator;
 
  constructor(page: Page) {
    this.page = page;
    // We define all our locators once in the constructor!
    this.usernameInput = page.getByTestId('username');
    this.passwordInput = page.getByTestId('password');
    this.loginButton = page.getByTestId('login-btn');
    this.headerText = page.locator('h2');
  }
 
  async goto() {
    await this.page.goto('https://practice.mycodeyatra.com/#/login');
  }
 
  async login(username: string, password: string) {
    // Actions use the predefined locators
    await this.usernameInput.fill(username);
    await this.passwordInput.fill(password);
    await this.loginButton.click();
  }
 
  async verifyLoginPage() {
    await expect(this.headerText).toHaveText('Sign In');
  }
}
```

### Writing the Test

Now, instead of duplicating locators in our test, we simply instantiate the Class and call its methods!

Create `tests/blog24_pom.spec.ts`:

```typescript
import { test, expect } from '@playwright/test';
import { LoginPage } from '../pages/LoginPage';
 
test.describe('Blog 24: Page Object Model', () => {
  test('Login using the Page Object Model', async ({ page }) => {
    // 1. Instantiate the Page Object
    const loginPage = new LoginPage(page);
    
    // 2. Interact with the page using high-level methods
    await loginPage.goto();
    await loginPage.verifyLoginPage();
    
    // 3. Perform the login action
    await loginPage.login('admin', 'admin123');
    
    // 4. Verify the outcome
    await expect(page).toHaveURL(/.*profile/);
    console.log('Successfully logged in using the Page Object Model!');
  });
});
```

### Execution Output

When you run `npx playwright test tests/blog24_pom.spec.ts`:

```
Running 1 test using 1 worker
 
Successfully logged in using the Page Object Model!
  OK   1 tests/blog24_pom.spec.ts:5:7 > Blog 24: Page Object Model > Login using the Page Object Model (1.0s)
 
  1 passed (2.3s)
```

### Conclusion

With the Page Object Model, if the developer changes `data-testid="username"` to `data-testid="email"`, you only have to update it in **one single file** (`LoginPage.ts`). All 50 tests will instantly inherit the fix. 

This pattern is non-negotiable for enterprise-level test automation.

In **Blog 25**, we will wrap up our Foundation series by introducing Playwright's **API Testing Capabilities**!
