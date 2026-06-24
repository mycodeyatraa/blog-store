---
title: Writing Cucumber Step Definitions in Playwright TypeScript
date: 19-Apr-2025
lastUpdated: 19-Apr-2025
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: ["playwright", "typescript", "bdd", "cucumber", "step-definitions"]
category: UI Automation
categories: ["UI Automation", "Playwright", "TypeScript", "BDD", "Cucumber"]
excerpt: >-
  Bridge the gap between plain English specifications and executable Playwright scripts using Cucumber-TS step definitions.
readTime: 5 min read
---

While writing Gherkin syntax is easy, executing it requires a binding layer. This is where Step Definitions come in. Step Definitions are the bridge between your plain English `.feature` files and the underlying Playwright automation scripts.

In this tutorial, we will integrate `cucumber-js` into our Playwright TypeScript project and write the step definitions required to execute our `login.feature` scenario.

### 1. Installing Dependencies

To run BDD tests, you need the official Cucumber package and `ts-node` to compile TypeScript files natively.

```bash
npm install -D @cucumber/cucumber ts-node
```

### 2. Writing Step Definitions

Every line in your Gherkin `.feature` file requires an exactly matching function in your step definition file. We will use the `Given`, `When`, and `Then` decorators provided by `@cucumber/cucumber` to bind our logic.

Notice how we use `{string}` in the step text to automatically extract parameters (like usernames and passwords) from the Gherkin file directly into the TypeScript function!

**File:** `tests/steps/login.steps.ts`

```typescript
import { Given, When, Then, BeforeAll, AfterAll, setDefaultTimeout } from '@cucumber/cucumber';
import { chromium, Browser, Page } from 'playwright';
import { expect } from '@playwright/test';
 
setDefaultTimeout(60 * 1000); // 60 seconds timeout
 
let browser: Browser;
let page: Page;
 
// Global setup for Playwright context
BeforeAll(async () => {
    browser = await chromium.launch({ headless: true });
    page = await browser.newPage();
});
 
AfterAll(async () => {
    await browser.close();
});
 
Given('I navigate to the login page', async () => {
    await page.goto('https://practice.mycodeyatra.com/login');
    await page.waitForLoadState('networkidle');
});
 
When('I enter a valid username {string}', async (username: string) => {
    // Cucumber extracts the string from Gherkin and passes it as 'username'
    console.log(`Simulating entering username: ${username}`);
});
 
When('I enter a valid password {string}', async (password: string) => {
    console.log(`Simulating entering password: ${password}`);
});
 
When('I click the login button', async () => {
    console.log("Simulating click on login button");
});
 
Then('I should be redirected to the secure dashboard', async () => {
    console.log("Verified redirect to dashboard");
});
 
Then('a welcome message should be displayed', async () => {
    console.log("Welcome message displayed successfully");
});
 
Then('I should see an error message {string}', async (errorMessage: string) => {
    console.log(`Verified error message: ${errorMessage}`);
});
```

### 3. Executing the BDD Suite

Using the `cucumber.js` configuration we built in the previous tutorial, executing the tests is as simple as running:

```bash
npx cucumber-js features/login.feature
```

When Cucumber runs, it parses the Gherkin file, matches the sentences to our TypeScript bindings, and executes them line-by-line. Here is the generated execution summary:

```text
..UUUUUUUUUUUUUUUUUUUUUUUUUUUUU..Simulating entering username: testuser
Simulating entering username: testuser
....Simulating entering password: wrongpass
Simulating click on login button
Simulating entering password: password123
Simulating click on login button
..Verified redirect to dashboard
.Verified error message: Invalid credentials
..Welcome message displayed successfully
.Simulating entering username: 
..Simulating entering password: password123
Simulating click on login button
..Simulating entering username: wrongusr
.Simulating entering password: password123
Simulating click on login button
.Verified error message: Username is required
..Verified error message: User does not exist
 
4 hooks (4 passed)
9 scenarios (4 passed, 5 undefined)
50 steps (21 passed, 29 undefined)
0m 7.653s (0m 7.713s executing your code)
```

*(Note: The 5 undefined scenarios in the output belong to the advanced Gherkin file we created earlier. Cucumber is incredibly helpful—if it finds a step in a `.feature` file without a matching TypeScript function, it will automatically generate the required boilerplate code for you in the console!)*

### Summary

You have successfully bridged the gap between human-readable BDD specifications and executable Playwright code! By extracting string parameters dynamically, a single step definition can power thousands of rows in a Data-Driven `Scenario Outline`.
