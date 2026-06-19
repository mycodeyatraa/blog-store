---
title: The Prompt Engineer: AI Test Generation
date: 15-Feb-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, ai, llms, gpt, prompt-engineering, cucumber]
category: Selenium TypeScript
categories: [Selenium TypeScript, AI in Testing]
excerpt: >-
  Enter the era of Prompt Engineering. Learn how to leverage Large Language Models (LLMs) to automatically generate Gherkin feature files and TypeScript step definitions directly from Jira tickets.
readTime: 4 min read
---

# The Prompt Engineer: AI Test Generation

Welcome to Phase 15. We have arrived at the bleeding edge of the QA Automation industry. 

For the last 14 phases, we have painstakingly written thousands of lines of TypeScript. We manually mapped CSS locators, manually wrote Cucumber Feature files, and manually defined Page Object Models.

In the era of Artificial Intelligence, this manual labor is slowly becoming obsolete. 

As a Senior QA Architect, your job is no longer just writing code. Your job is now **Prompt Engineering**: instructing AI models to write the code for you.

---

## 1. The Era of Large Language Models (LLMs)

Large Language Models (like OpenAI's GPT-4, Google's Gemini, or Anthropic's Claude) have ingested billions of lines of code from open-source repositories. 

They already know how to write a Selenium WebDriver test in TypeScript. They already know the Cucumber syntax. 

Instead of opening your IDE and typing `import { Builder } from 'selenium-webdriver';`, your workflow now begins with a **Prompt**.

---

## 2. Generating Gherkin Features

The first step of any automation task is writing the Behavior-Driven Development (BDD) scenarios.

Imagine you receive a Jira ticket:
> *"Users should be able to log in with valid credentials. If they use an invalid password, they should see an error message."*

Instead of writing the `.feature` file manually, you provide the Jira ticket to an LLM with the following prompt:

**Prompt:**
> *"Act as a Senior QA Automation Engineer. Convert the following Jira ticket into a Gherkin Feature file. Ensure you use standard Given/When/Then syntax and include a Scenario Outline for the invalid credentials."*

**AI Output:**

```gherkin
Feature: User Authentication
  Scenario: Successful Login
    Given the user navigates to the login page
    When the user enters valid credentials "admin" and "password123"
    And clicks the login button
    Then they should be redirected to the secure dashboard
  Scenario Outline: Unsuccessful Login
    Given the user navigates to the login page
    When the user enters valid username "<username>" and invalid password "<password>"
    And clicks the login button
    Then they should see the error message "<error_msg>"
    Examples:
      | username | password | error_msg               |
      | admin    | wrong1   | Invalid Password        |
      | admin    | blank    | Password cannot be null |
```

In 5 seconds, the AI has generated perfectly formatted Gherkin syntax, including edge cases you might not have considered!

---

## 3. Generating TypeScript Step Definitions

Now that you have the Gherkin file, you need the TypeScript code to execute it.

You feed the generated Gherkin back into the LLM with this prompt:

**Prompt:**
> *"Generate the cucumber-js TypeScript step definitions for this feature file. Use an existing `LoginPage` Page Object Model. Assume `driver` is globally available. Do not implement the `LoginPage` class itself, just the steps."*

**AI Output:**

```typescript
import { Given, When, Then } from '@cucumber/cucumber';
import { LoginPage } from '../../pages/LoginPage';
import { DashboardPage } from '../../pages/DashboardPage';
import { expect } from 'chai';
Given('the user navigates to the login page', async function () {
  const loginPage = new LoginPage(this.driver);
  await loginPage.navigate();
});
When('the user enters valid credentials {string} and {string}', async function (username, password) {
  const loginPage = new LoginPage(this.driver);
  await loginPage.enterCredentials(username, password);
});
// ... remaining steps generated instantly ...
```

---

## 4. The Architect's Review

This is where your expertise as a Senior QA Architect becomes critical. 

**The AI is an assistant, not a replacement.**

You must review the generated code. Did the AI use hardcoded `Thread.sleep()` instead of explicit `WebDriverWait`? Did it fail to handle a Shadow DOM element? Did it ignore your custom Winston logging strategy?

You take the generated template, refine it, fix the architectural mistakes, and commit it. What used to take 2 hours now takes 15 minutes.

## Conclusion

AI Test Generation drastically accelerates the initial drafting of automation code. By mastering Prompt Engineering, you can transform Jira tickets into executable TypeScript in seconds.

But what happens when the UI changes? What happens when a developer changes an `id="submit-btn"` to `id="login-btn"`, and all your tests instantly fail?

In the past, you had to manually update the Page Object Model. 

In our next tutorial, we will explore **AI Locator Healing**, where the automation framework uses Machine Learning to fix broken CSS selectors *at runtime*, before the test even fails!
