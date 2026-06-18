---
title: Mastering the Lifecycle: Hooks in Cucumber-TS
date: 13-Jan-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, bdd, cucumber, hooks, lifecycle, screenshots]
category: Selenium TypeScript
categories: [Selenium TypeScript, BDD & Cucumber Framework]
excerpt: >-
  Control your Selenium WebDriver lifecycle, manage browser sessions, and dynamically capture failure screenshots using Cucumber's Before and After hooks.
readTime: 5 min read
---

# Mastering the Lifecycle: Hooks in Cucumber-TS

In our previous tutorial, we wrote Step Definitions to drive the browser. However, we had a major architectural problem: where does the `driver` variable come from?

In a standard Jest framework, we use `beforeAll()` to instantiate `new Builder().forBrowser("chrome").build()`, and `afterAll()` to call `driver.quit()`. 

Cucumber does not use Jest. It uses its own execution engine, and therefore, it has its own lifecycle methods called **Hooks**.

---

## 1. The Four Core Hooks

Cucumber provides four primary decorators to manage your test execution lifecycle:

1. **`BeforeAll`**: Runs exactly once before the entire Cucumber test suite starts (e.g., establishing a database connection).
2. **`Before`**: Runs before *every single Scenario* in every feature file (e.g., launching a fresh Chrome browser).
3. **`After`**: Runs after *every single Scenario* (e.g., taking a screenshot and quitting the browser).
4. **`AfterAll`**: Runs exactly once after all tests have finished executing.

---

## 2. Implementing the Global Hook

Let's create the file that will actually instantiate our Selenium WebDriver!

Create `tests/bdd/support/hook.ts`:

```typescript
import { BeforeAll, AfterAll, Before, After, Status } from "@cucumber/cucumber";
import { Builder, WebDriver } from "selenium-webdriver";
// We export the driver so our Step Definitions can import and use it!
export let driver: WebDriver;
// Runs ONCE before the entire test suite starts
BeforeAll(async function () {
  console.log("[Cucumber Hook] BeforeAll: Initializing global resources...");
});
// Runs before EVERY SINGLE Scenario in every feature file
Before(async function (scenario) {
  console.log(`[Cucumber Hook] Before Scenario: ${scenario.pickle.name}`);
  // Initialize a fresh browser for every scenario to ensure state isolation
  driver = await new Builder().forBrowser("chrome").build();
  await driver.manage().window().maximize();
});
// Runs after EVERY SINGLE Scenario
After(async function (scenario) {
  console.log(`[Cucumber Hook] After Scenario: ${scenario.pickle.name}`);
  // If the scenario failed, let's take a screenshot!
  if (scenario.result?.status === Status.FAILED) {
    console.error(`[Cucumber Hook] Scenario Failed! Taking Screenshot...`);
    const screenshot = await driver.takeScreenshot();
    // Attach the screenshot to the Cucumber HTML Report
    this.attach(Buffer.from(screenshot, "base64"), "image/png");
  }
  // Always quit the driver to prevent zombie processes
  if (driver) {
    await driver.quit();
  }
});
// Runs ONCE after the entire test suite finishes
AfterAll(async function () {
  console.log("[Cucumber Hook] AfterAll: Tearing down global resources...");
});
```

### Breaking down the Hook:
1. **`export let driver: WebDriver;`**: By exporting this variable, all of our individual `login.steps.ts` or `checkout.steps.ts` files can simply `import { driver } from "../support/hook"` and immediately have access to the browser session!
2. **`scenario.pickle.name`**: The `scenario` object passed into the `Before` and `After` hooks contains all the metadata about the currently executing Gherkin block. `pickle.name` is the actual string name of the scenario!
3. **`this.attach()`**: If the scenario fails, we capture a base64 screenshot from Selenium and attach it directly to the HTML report we configured in `cucumber.js`!

## Conclusion

Hooks are the invisible backbone of your BDD automation architecture. They ensure that every Gherkin scenario starts with a clean slate and that no Chrome processes are left running in memory.

However, exporting a global `driver` variable from a hook file works fine for basic sequential execution, but what happens when we want to run our tests in **Parallel**? Global variables will overwrite each other, causing absolute chaos!

In our next tutorial, we will explore **Scenario Context (The World Object)** to learn how to properly isolate browser state for parallel BDD execution!
