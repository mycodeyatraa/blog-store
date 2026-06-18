---
title: The Glue Code: Writing Cucumber Step Definitions
date: 12-Jan-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, bdd, cucumber, step-definitions, cucumber-expressions]
category: Selenium TypeScript
categories: [Selenium TypeScript, BDD & Cucumber Framework]
excerpt: >-
  Connect your plain-English Gherkin scenarios to your Selenium automation by writing TypeScript Step Definitions and utilizing dynamic Cucumber Expressions.
readTime: 4 min read
---

# The Glue Code: Writing Cucumber Step Definitions

In our previous tutorials, we wrote beautiful Gherkin feature files, but when we ran the Cucumber execution engine, it complained that the steps were "Undefined."

Cucumber reads a sentence like `Given the user navigates to the login page`, but it has no idea what that actually means. Does it need to open Chrome? Firefox? Does it need to hit an API? 

To give Cucumber instructions, we must write **Step Definitions**. This is the "Glue Code" that binds the English sentences to our TypeScript automation.

---

## 1. Importing the Cucumber Hooks

Step Definitions are simply TypeScript functions wrapped in decorators provided by the `@cucumber/cucumber` package.

Create a new file at `tests/bdd/steps/login.steps.ts`:

```typescript
import { Given, When, Then } from "@cucumber/cucumber";
import { expect } from "expect";
// The string inside the decorator MUST perfectly match the Gherkin sentence!
Given('the user navigates to the MyCodeYatra login page', async function () {
  console.log("[Selenium] Navigating to https://practice.mycodeyatra.com/login");
  // await driver.get("https://practice.mycodeyatra.com/login");
});
When('clicks the login button', async function () {
  console.log("[Selenium] Clicking Login Button");
  // await driver.findElement(By.id("login-btn")).click();
});
```

Whenever Cucumber parses the `.feature` file and reads the sentence `Given the user navigates to the MyCodeYatra login page`, it will scan all your `.ts` files looking for a `Given` function that has the *exact same string*. Once it finds a match, it executes the code block inside!

---

## 2. Dynamic Parameters (Cucumber Expressions)

What happens if we want to log in with different users? Do we have to write 50 different `When` functions for 50 different usernames? 

No! We use **Cucumber Expressions** to pass variables directly from the English text into our TypeScript function arguments.

Look at our Gherkin file from Tutorial 1:
`When the user enters the username "admin" and password "password123"`

Now, look at how we write the Step Definition:

```typescript
When('the user enters the username {string} and password {string}', async function (username, password) {
  // Cucumber automatically parses "admin" from the feature file and passes it to the `username` argument!
  console.log(`[Selenium] Typing username: ${username}`);
  console.log(`[Selenium] Typing password: ${password}`);
  // await driver.findElement(By.id("username")).sendKeys(username);
  // await driver.findElement(By.id("password")).sendKeys(password);
});
```

By replacing the hardcoded text with `{string}`, Cucumber knows to extract whatever is inside the quotation marks in the `.feature` file and pass it as an argument to our async function! 

You can also use `{int}` for numbers, or `{float}` for decimals!

---

## 3. The `Then` Step (Assertions)

The `Then` step is reserved exclusively for assertions. This is where we validate that the `When` action actually worked.

```typescript
Then('the user should be redirected to the secure dashboard', async function () {
  console.log("[Selenium] Asserting Dashboard URL is present");
  // const currentUrl = await driver.getCurrentUrl();
  // expect(currentUrl).toContain("/dashboard");
});
```

Because Cucumber doesn't come with its own assertion library, we simply import `expect` (or Chai, if you prefer) and use it exactly as we did in our standard Jest framework.

## Conclusion

Step definitions are the bridge between Product Managers (who write the English) and Automation Engineers (who write the TypeScript). 

However, you might have noticed something missing from our scripts today. Where did `driver` come from? We didn't initialize `new Builder().forBrowser('chrome').build()` anywhere!

In our Jest framework, we used `beforeAll` and `afterAll`. In Cucumber, the architecture is entirely different. 

In our next tutorial, we will explore **Cucumber Hooks** and learn how to manage our Selenium WebDriver lifecycle seamlessly across thousands of BDD scenarios!
