---
title: Mastering State Management: Jest Lifecycle & Hooks
date: 24-Nov-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, jest, lifecycle-hooks, state-management]
category: Selenium TypeScript
categories: [Selenium TypeScript, Architecture & Patterns]
excerpt: >-
  When should the WebDriver instantiate? How do we capture screenshots on failure? Learn how to orchestrate Jest's beforeAll, beforeEach, afterEach, and afterAll hooks.
readTime: 5 min read
---

# Mastering State Management: Jest Lifecycle & Hooks

As you begin organizing your automation suite into the Page Object Model and Factory Patterns, you'll encounter a crucial architectural challenge: **State Management**.

When should the WebDriver instantiate? Should it happen before every single test case, or just once per file? What if a test fails—how do we capture a screenshot?

In Jest, these questions are answered using **Lifecycle Hooks**.

---

## 1. The Four Core Hooks

Jest provides four primary lifecycle methods that wrap around your `it()` test cases:

1. `beforeAll()`: Runs exactly once before any tests in the file. Perfect for database connections or opening a single Browser instance.
2. `beforeEach()`: Runs before *every* test. Perfect for navigating to a fresh URL or clearing cookies.
3. `afterEach()`: Runs after *every* test. Perfect for taking failure screenshots.
4. `afterAll()`: Runs exactly once after all tests finish. Essential for `driver.quit()` to prevent memory leaks!

---

## 2. Implementing Hooks with Selenium WebDriver

Let's look at how we orchestrate these hooks with Selenium to ensure our tests remain completely independent and flake-free.

Create `tests/jest_lifecycle.test.ts`:

```typescript
import { WebDriver } from "selenium-webdriver";
import { AdvancedDriverFactory } from "./utils/advancedDriverFactory";
describe("Architecture Phase - Jest Lifecycle & Hooks", () => {
  let driver: WebDriver;
  // 1. Executes ONCE before ALL tests
  beforeAll(async () => {
    console.log("[beforeAll] -> Initializing Global WebDriver Connection...");
    driver = await AdvancedDriverFactory.getDriver();
  });
  // 2. Executes BEFORE EACH individual test (it block)
  beforeEach(async () => {
    console.log("[beforeEach] -> Navigating to fresh state...");
    await driver.get("https://practice.mycodeyatra.com/#/form-practice");
    // Ensure the browser state is clean for the upcoming test!
    await driver.manage().deleteAllCookies();
  });
  // 3. Executes AFTER EACH individual test (it block)
  afterEach(async () => {
    console.log("[afterEach] -> Checking for test failure to take screenshot...");
    // Future lesson: We will implement 'jest-circus' to detect failures here!
  });
  // 4. Executes ONCE after ALL tests
  afterAll(async () => {
    console.log("[afterAll] -> Quitting WebDriver and closing connections...");
    if (driver) {
      await driver.quit();
    }
  });
  it("Test Case 1: Validate Title", async () => {
    console.log("-> Executing Test Case 1...");
    const title = await driver.getTitle();
    expect(title).toBeDefined();
  });
  it("Test Case 2: Validate URL", async () => {
    console.log("-> Executing Test Case 2...");
    const url = await driver.getCurrentUrl();
    expect(url).toContain("form-practice");
  });
});
```

---

## 3. Visualizing the Execution Order

When we run this script, observe the console logs. The execution flow creates a "sandwich" around our tests:

```text
> jest tests/jest_lifecycle.test.ts
  console.log
    [beforeAll] -> Initializing Global WebDriver Connection...
  console.log
    [beforeEach] -> Navigating to fresh state...
  console.log
    -> Executing Test Case 1...
  console.log
    [afterEach] -> Checking for test failure to take screenshot...
  console.log
    [beforeEach] -> Navigating to fresh state...
  console.log
    -> Executing Test Case 2...
  console.log
    [afterEach] -> Checking for test failure to take screenshot...
  console.log
    [afterAll] -> Quitting WebDriver and closing connections...
 PASS  tests/jest_lifecycle.test.ts (8.221 s)
  Architecture Phase - Jest Lifecycle & Hooks
    √ Test Case 1: Validate Title (210 ms)
    √ Test Case 2: Validate URL (180 ms)
```

## Conclusion

By strategically placing logic inside Jest's Lifecycle Hooks, we ensure that:
1. **Tests are Isolated:** `beforeEach` clears state, preventing Test 2 from failing due to leftover data from Test 1.
2. **Resources are Conserved:** `beforeAll` ensures we don't waste 10 seconds spawning a new browser for every single `it` block.
3. **No Memory Leaks:** `afterAll` guarantees the ChromeDriver process is killed.

This officially concludes **Phase 3: Advanced Architecture & Design Patterns!** 

In **Phase 4**, we will explore **Framework Utilities**, starting with building a robust Utility Framework.
