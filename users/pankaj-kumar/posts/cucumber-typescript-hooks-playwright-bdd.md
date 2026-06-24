---
title: Mastering Cucumber Hooks in Playwright TypeScript
date: 21-Apr-2025
lastUpdated: 21-Apr-2025
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: ["playwright", "typescript", "bdd", "cucumber", "hooks"]
category: UI Automation
categories: ["UI Automation", "Playwright", "TypeScript", "BDD", "Cucumber"]
excerpt: >-
  Learn how to use Cucumber's global and tagged lifecycle hooks to manage browser contexts, database connections, and test reporting.
readTime: 5 min read
---

In enterprise automation, testing is never just about executing steps. You need to connect to databases, spin up browser contexts, intercept network traffic, and capture screenshots when tests fail. 

Cucumber **Hooks** are blocks of code that run at specific points in the testing lifecycle. They are the architectural backbone of a robust Playwright-BDD framework.

### Global vs Scenario Hooks

Hooks come in two main flavors:
1.  **Global Lifecycle (`BeforeAll`, `AfterAll`)**: These run exactly once before the entire test suite starts, and once after everything finishes. They are ideal for heavy setup tasks like authenticating a user globally or bootstrapping a database connection.
2.  **Scenario Lifecycle (`Before`, `After`)**: These run before and after *every single scenario*. They are perfect for clearing cookies, navigating to base URLs, or taking a screenshot on failure.

### Implementing Hooks in TypeScript

Let's look at an implementation that handles both global setup and scenario-specific tear-down:

**File:** `tests/steps/hooks.ts`

```typescript
import { BeforeAll, AfterAll, Before, After, Status } from '@cucumber/cucumber';
 
// Runs exactly once before the ENTIRE test suite
BeforeAll(async function () {
    console.log('[Hook: BeforeAll] Bootstrapping Database Connections...');
    console.log('[Hook: BeforeAll] Launching Playwright Browser Context...');
});
 
// Runs exactly once after the ENTIRE test suite
AfterAll(async function () {
    console.log('[Hook: AfterAll] Closing Browser Context...');
    console.log('[Hook: AfterAll] Tearing down Database Connections...');
});
 
// Runs before EVERY single scenario
Before(async function (scenario) {
    console.log(`[Hook: Before] Starting Scenario: ${scenario.pickle.name}`);
});
 
// Runs after EVERY single scenario
After(async function (scenario) {
    // We can conditionally check if the scenario failed!
    if (scenario.result?.status === Status.FAILED) {
        console.log(`[Hook: After] SCENARIO FAILED! Capturing Screenshot for ${scenario.pickle.name}...`);
        // await page.screenshot({ path: `reports/${scenario.pickle.name}.png` });
    } else {
        console.log(`[Hook: After] Scenario ${scenario.pickle.name} executed successfully.`);
    }
});
```

### Tagged Hooks

Sometimes you need a setup step that only applies to specific tests (like injecting massive amounts of mock data). Instead of running it before *every* test, you can bind a hook to a specific Gherkin `@tag`:

```typescript
// Tagged Hook: Only runs before scenarios tagged with @data-driven
Before({ tags: '@data-driven' }, async function () {
    console.log('[Tagged Hook: @data-driven] Injecting test data into scenario context...');
});
```

### Execution Results

When we execute our test suite, Cucumber orchestrates the lifecycle natively. Notice how the global `BeforeAll` runs first, followed by the scenario-specific `Before`, and the conditionally executed `[Tagged Hook]`:

```text
[Hook: BeforeAll] Bootstrapping Database Connections...
[Hook: BeforeAll] Launching Playwright Browser Context...
 
[Hook: Before] Starting Scenario: Applying promotional discount codes
[Tagged Hook: @data-driven] Injecting test data into scenario context...
[Setup] User authenticated successfully.
[Setup] Cart cleared.
Added Laptop at $1000
Applying Promo Code: SAVE10
Asserting final price is $900
Asserting discount message: 10% discount applied!
[Hook: After] Scenario Applying promotional discount codes executed successfully.
 
...
 
[Hook: AfterAll] Closing Browser Context...
[Hook: AfterAll] Tearing down Database Connections...
 
8 hooks (8 passed)
9 scenarios (9 passed)
71 steps (71 passed)
0m 10.218s
```

### Summary

Hooks are essential for maintaining clean step definitions. By abstracting setup and teardown logic (like Playwright browser instantiation and screenshot capture) into central hook files, your step definitions remain focused purely on testing business logic!
