---
title: Surgical Execution: Filtering Tests with Cucumber Tags
date: 15-Jan-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, bdd, cucumber, tags, testing, execution]
category: Selenium TypeScript
categories: [Selenium TypeScript, BDD & Cucumber Framework]
excerpt: >-
  Learn how to categorize your BDD scenarios using Cucumber tags (@smoke, @regression) to run targeted test executions from the command line.
readTime: 4 min read
---

# Surgical Execution: Filtering Tests with Cucumber Tags

When your team writes hundreds of BDD scenarios, executing all of them takes a significant amount of time. 

If a developer pushes a quick hotfix, they don't want to wait 45 minutes for the entire regression suite to run. They just want to run a quick 2-minute "Smoke Test" to ensure the core functionality (like Login and Checkout) is still working.

How do we tell Cucumber to *only* run the Smoke Tests and ignore everything else?

We use **Tags**.

---

## 1. Tagging Scenarios in Gherkin

Tags in Cucumber are simply words prefixed with an `@` symbol. They can be placed above a `Feature` (applying to all scenarios inside it) or above a specific `Scenario`.

Let's look at `tests/bdd/features/authentication/login.feature`:

```gherkin
@auth @regression
Feature: User Authentication
  @smoke @critical
  Scenario: Valid Login
    Given the user navigates to the login page
    When the user enters valid credentials
    Then the user should be redirected to the dashboard
  @negative
  Scenario: Invalid Login
    Given the user navigates to the login page
    When the user enters invalid credentials
    Then an error message should appear
```

In this file:
- Both scenarios inherit the `@auth` and `@regression` tags from the Feature level.
- Only the "Valid Login" scenario has the `@smoke` and `@critical` tags.
- Only the "Invalid Login" scenario has the `@negative` tag.

---

## 2. Executing Tags from the CLI

Now that our Gherkin is tagged, we can use the Cucumber Command Line Interface (CLI) to filter execution.

To run *only* scenarios tagged with `@smoke`:

```bash
npx cucumber-js --tags "@smoke"
```
*(This will run "Valid Login" but completely skip "Invalid Login").*

To run everything *except* smoke tests:

```bash
npx cucumber-js --tags "not @smoke"
```

To run scenarios that have *both* `@auth` AND `@smoke`:

```bash
npx cucumber-js --tags "@auth and @smoke"
```

To run scenarios that are *either* `@smoke` OR `@regression`:

```bash
npx cucumber-js --tags "@smoke or @regression"
```

---

## 3. Configuring NPM Scripts for CI/CD

Developers shouldn't have to memorize complex CLI tag expressions. We should bake these commands into our `package.json` so they can be easily executed locally or triggered by a Jenkins/GitHub Actions pipeline!

Update your `package.json`:

```json
{
  "scripts": {
    "test:bdd": "cucumber-js",
    "test:bdd:smoke": "cucumber-js --tags '@smoke'",
    "test:bdd:regression": "cucumber-js --tags '@regression'",
    "test:bdd:auth": "cucumber-js --tags '@auth'"
  }
}
```

Now, when a developer pushes a hotfix, the CI pipeline can simply execute:

```bash
npm run test:bdd:smoke
```

## Conclusion

Tags are an incredibly powerful tool for orchestrating large test suites, allowing you to slice and dice your execution strategy based on the specific needs of your CI/CD pipeline.

We have now covered Feature Files, Step Definitions, Hooks, Scenario Context, and Tags. We have successfully built a robust Cucumber architecture!

In our final tutorial of Phase 10, we will explore an alternative approach for teams heavily invested in the Jest ecosystem: **Jest-Cucumber**. This library allows you to write Gherkin syntax without abandoning the Jest runner!
