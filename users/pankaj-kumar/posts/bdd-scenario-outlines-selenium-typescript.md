---
title: Data-Driven BDD: Mastering Scenario Outlines
date: 15-Jan-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, bdd, cucumber, scenario-outlines, data-driven, testing]
category: Selenium TypeScript
categories: [Selenium TypeScript, BDD]
excerpt: >-
  Keep your feature files DRY. Learn how to construct data-driven Scenario Outlines using Examples tables to multiply your test coverage without duplicating code.
readTime: 4 min read
---

# Data-Driven BDD: Mastering Scenario Outlines

When writing Behavior-Driven Development (BDD) tests using Cucumber, you will often encounter a situation where you need to execute the exact same test logic multiple times, but with different input data.

For example, testing a Login page with:
1. Valid User / Valid Password
2. Invalid User / Valid Password
3. Valid User / Invalid Password
4. Empty User / Empty Password

If you write a separate `Scenario` for each of these permutations, your `.feature` file will become hundreds of lines long, violating the DRY (Don't Repeat Yourself) principle.

The solution is the **Scenario Outline**.

---

## 1. What is a Scenario Outline?

A `Scenario Outline` allows you to write the Gherkin steps exactly once, and then provide a table of `Examples`. Cucumber will automatically iterate over the table, executing the Scenario once for every row.

Let's look at how to structure this.

### The Problem: Repetitive Scenarios

```gherkin
Feature: Login Validation
  Scenario: Invalid Username
    Given I navigate to the login page
    When I enter "baduser" and "Secret123"
    And I click submit
    Then I should see the error "Invalid Credentials"
  Scenario: Invalid Password
    Given I navigate to the login page
    When I enter "admin" and "badpassword"
    And I click submit
    Then I should see the error "Invalid Credentials"
  # ... 5 more scenarios ...
```

### The Solution: Scenario Outline

```gherkin
Feature: Login Validation
  Scenario Outline: Unsuccessful Login Attempts
    Given I navigate to the login page
    When I enter "<username>" and "<password>"
    And I click submit
    Then I should see the error "<error_message>"
    Examples:
      | username | password    | error_message       |
      | baduser  | Secret123   | Invalid Credentials |
      | admin    | badpassword | Invalid Credentials |
      |          | Secret123   | Username is required|
      | admin    |             | Password is required|
```

Notice the use of angle brackets `<var>`. These act as variable placeholders.

---

## 2. Implementing Step Definitions in TypeScript

The beauty of the `Scenario Outline` is that your TypeScript step definitions do not need to change at all! 

Because Cucumber automatically injects the values from the `Examples` table into the Gherkin strings, your standard Regex or Cucumber Expressions will catch them natively.

```typescript
import { Given, When, Then } from '@cucumber/cucumber';
import { LoginPage } from '../pages/LoginPage';
import { expect } from 'chai';
Given('I navigate to the login page', async function () {
  const loginPage = new LoginPage(this.driver);
  await loginPage.navigate();
});
// The {string} placeholders will catch the injected variables from the table!
When('I enter {string} and {string}', async function (username, password) {
  const loginPage = new LoginPage(this.driver);
  await loginPage.enterCredentials(username, password);
});
Then('I should see the error {string}', async function (expectedError) {
  const loginPage = new LoginPage(this.driver);
  const actualError = await loginPage.getErrorMessage();
  expect(actualError).to.equal(expectedError);
});
```

---

## 3. Data Tables vs. Scenario Outlines

A common point of confusion for Junior QA Engineers is the difference between a `Data Table` and a `Scenario Outline`.

**Data Table:**
Used to pass a list or matrix of data to a *single* step within a single execution.

```gherkin
Scenario: Create Multiple Users
  Given I am logged in as an Admin
  When I create the following users:
    | John | Admin |
    | Jane | User  |
  Then I should see a success message
```

**Scenario Outline:**
Used to execute the *entire scenario* multiple times from top to bottom.

```gherkin
Scenario Outline: Verify User Permissions
  Given I am logged in as a "<role>"
  When I attempt to access the dashboard
  Then I should see the message "<message>"
  Examples:
    | role  | message        |
    | Admin | Welcome Admin  |
    | Guest | Access Denied  |
```

## Conclusion

The `Scenario Outline` is one of the most powerful tools in the BDD ecosystem. It allows you to transform your Gherkin files from simple scripts into massive, data-driven test suites while keeping your TypeScript code perfectly DRY.

Now that we have fully mastered the BDD layer, it is time to move on to Phase 11, where we will completely leave the UI and learn how to perform hardcore Enterprise Validation (Databases, Kafka, Email, and PDFs!).
