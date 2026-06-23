---
title: "Dependency Injection with Playwright Fixtures"
date: "2025-03-16"
description: "Learn how to use Playwright's built-in fixture system as a Dependency Injection (DI) container to eliminate boilerplates and coupling in Page Object Model (POM)."
tags: ["Playwright", "TypeScript", "Dependency Injection", "Design Patterns"]
---

Welcome to Blog 33 of the **Playwright TypeScript Mastery Series**!

When building a large-scale test automation framework, the Page Object Model (POM) is the standard pattern for organizing locators and actions. However, as the suite grows, you often end up writing repetitive instantiation code in every single test:

```typescript
test('Traditional Instantiation', async ({ page }) => {
  const loginPage = new LoginPage(page);
  const dashboardPage = new DashboardPage(page);
 
  await loginPage.login();
  await dashboardPage.verifyDashboard();
});
```

This traditional approach introduces **tight coupling** and **boilerplate code**. If the constructor of `LoginPage` changes tomorrow, you will have to update it in dozens of test files.

Today, we will solve this using **Dependency Injection (DI)** powered by **Playwright Fixtures**.

---

### What is Dependency Injection?

Dependency Injection is a design pattern where an object receives other objects that it depends on, rather than creating them itself. 

In Playwright, **Fixtures** act as the dependency injection container. Instead of instantiating page objects inside the test, you declare them as arguments in your test function. Playwright then automatically instantiates and injects them at runtime.

---

### Step 1: Define the Page Objects

Let's create two simple Page Object classes: `LoginPage` and `DashboardPage`.

```typescript
class LoginPage {
  constructor(public page: any) {}
 
  async login() {
    console.log('LoginPage: Executing login workflow...');
  }
}
 
class DashboardPage {
  constructor(public page: any) {}
 
  async verifyDashboard() {
    console.log('DashboardPage: Verifying dashboard state...');
  }
}
```

---

### Step 2: Set Up the DI Container (Extending the Test)

To make Playwright aware of our page objects, we extend the base test using `base.extend<T>()`. This acts as our DI registry:

```typescript
import { test as base, expect } from '@playwright/test';
 
// Define the types of our dependencies (fixtures)
type AppFixtures = {
  loginPage: LoginPage;
  dashboardPage: DashboardPage;
};
 
// Extend base test to inject page objects automatically
export const test = base.extend<AppFixtures>({
  loginPage: async ({ page }, use) => {
    // Instantiation happens HERE in a single place!
    const loginPage = new LoginPage(page);
    await use(loginPage);
  },
  dashboardPage: async ({ page }, use) => {
    const dashboardPage = new DashboardPage(page);
    await use(dashboardPage);
  }
});
```

---

### Step 3: Write Tests with Injected Dependencies

Now, look how clean and boilerplate-free our test file becomes. We simply destructure the dependencies we need in the test parameters:

```typescript
import { test } from './fixtures'; // import our extended test
 
test.describe('Blog 33: Dependency Injection', () => {
 
  test('DI through Fixtures', async ({ loginPage, dashboardPage }) => {
    console.log('Test started with injected dependencies.');
    
    await loginPage.login();
    await dashboardPage.verifyDashboard();
    
    console.log('Test completed successfully via Dependency Injection!');
  });
 
});
```

Notice that we did not write `new LoginPage(page)` anywhere in the spec file! Playwright handle the lifecycle, instantiates them on-demand, and cleans them up after the test completes.

---

### Step 4: Run the Test

Execute the test using the Playwright CLI:

```
npx playwright test tests/blog33_di.spec.ts --headed
```

**Output:**

```
Running test against environment...
LoginPage: Executing login workflow...
DashboardPage: Verifying dashboard state...
Test completed successfully via Dependency Injection!
  1 passed (3.5s)
```

---

### Benefits of Dependency Injection in Playwright

1. **Dry Code (Don't Repeat Yourself)**: Page Objects are instantiated in one central place rather than in every test file.
2. **Lazy Initialization**: Playwright only instantiates a fixture if the test function explicitly requests it as a parameter.
3. **Decoupled Architecture**: If the constructor configuration changes, you only update it once in the fixture definition file.
4. **Automatic Cleanups**: Playwright handles setup and teardown for each fixture automatically.

In the next blog, we will step into **Phase 5** and explore **API Testing** in Playwright using `APIRequestContext`!
