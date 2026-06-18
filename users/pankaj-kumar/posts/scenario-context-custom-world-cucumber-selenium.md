---
title: Eliminating Global Variables: The Cucumber World Object
date: 14-Jan-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, bdd, cucumber, custom-world, parallel-execution, state]
category: Selenium TypeScript
categories: [Selenium TypeScript, BDD & Cucumber Framework]
excerpt: >-
  Enable safe parallel execution of BDD suites by replacing global variables with a Custom Cucumber World class to perfectly isolate Selenium WebDriver instances.
readTime: 5 min read
---

# Eliminating Global Variables: The Cucumber World Object

In our previous tutorial, we exported a global `driver` variable from our `hook.ts` file so that our Step Definitions could import it and interact with the browser.

```typescript
// hook.ts
export let driver: WebDriver;
```

This works perfectly... **if** you run your tests sequentially (one at a time). 

But what happens when we enable **Parallel Execution** to run 10 scenarios simultaneously across a Selenium Grid? 
- Scenario A launches Chrome and assigns the browser instance to `driver`.
- Scenario B launches Firefox and *overwrites* the global `driver` variable.
- Scenario A attempts to click a button, but it sends the command to Firefox instead of Chrome, crashing the suite!

To run tests in parallel, we must completely eliminate global variables. We achieve this using **The Cucumber World Object**.

---

## 1. Creating a Custom World

In Cucumber, every single Scenario gets its own strictly isolated context, referred to as the `World`. By default, the World is an empty object, but we can extend it to hold our `WebDriver` instance and any other shared data (like API tokens or User IDs) that we want to pass between steps!

Create `tests/bdd/support/world.ts`:

```typescript
import { setWorldConstructor, World, IWorldOptions } from "@cucumber/cucumber";
import { Builder, WebDriver } from "selenium-webdriver";
// Extend the default Cucumber World
export class CustomWorld extends World {
  public driver?: WebDriver;
  // A place to store dynamic data between steps (e.g., extracting an Order ID)
  public sharedData: { [key: string]: any } = {};
  constructor(options: IWorldOptions) {
    super(options);
  }
  // Helper method to safely get the driver
  public getDriver(): WebDriver {
    if (!this.driver) {
      throw new Error("WebDriver is not initialized. Check your Before hook.");
    }
    return this.driver;
  }
}
// Tell Cucumber to use our CustomWorld instead of the default World object
setWorldConstructor(CustomWorld);
```

---

## 2. Refactoring Hooks to use `this`

Now that we have a Custom World, we no longer need to export a global variable. Instead, we assign the browser directly to `this.driver`. Because Cucumber instantiates a fresh `CustomWorld` class for every scenario, `this` is perfectly isolated!

Update your `hook.ts`:

```typescript
import { Before, After } from "@cucumber/cucumber";
import { Builder } from "selenium-webdriver";
import { CustomWorld } from "./world";
Before(async function (this: CustomWorld) {
  // We assign the driver to the isolated World context, NOT a global variable!
  this.driver = await new Builder().forBrowser("chrome").build();
});
After(async function (this: CustomWorld) {
  if (this.driver) {
    await this.driver.quit();
  }
});
```

---

## 3. Refactoring Step Definitions to use `this`

Finally, we update our Step Definitions. We no longer `import { driver }`. Instead, we simply access `this.driver`!

Update `login.steps.ts`:

```typescript
import { Given, When, Then } from "@cucumber/cucumber";
import { CustomWorld } from "../support/world";
// IMPORTANT: Do NOT use arrow functions () => {} in Step Definitions! 
// Arrow functions destroy the `this` binding in JavaScript/TypeScript.
// Always use standard `function () {}` syntax.
Given('the user navigates to the MyCodeYatra login page', async function (this: CustomWorld) {
  const driver = this.getDriver(); // Safely retrieve the isolated driver
  console.log("[Selenium] Navigating via isolated driver!");
  // await driver.get("https://practice.mycodeyatra.com/login");
});
When('clicks the login button', async function (this: CustomWorld) {
  const driver = this.getDriver();
  console.log("[Selenium] Clicking Login Button via isolated driver!");
  // await driver.findElement(By.id("login-btn")).click();
});
```

## Conclusion

By adopting the **Custom World** pattern, you have completely eliminated global state leakage. You can now execute 50 scenarios in parallel across a Selenium grid, and every single scenario will safely maintain its own perfectly isolated `WebDriver` instance!

But what if you only want to run a specific subset of those 50 scenarios? Say, only the scenarios marked for "Smoke Testing"?

In our next tutorial, we will dive deep into **Tags**, allowing us to execute surgical, targeted test runs directly from the command line!
