---
title: Filtering Execution using Cucumber Tags in Playwright
date: 23-Apr-2025
lastUpdated: 23-Apr-2025
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: ["playwright", "typescript", "bdd", "cucumber", "tags", "cli"]
category: UI Automation
categories: ["UI Automation", "Playwright", "TypeScript", "BDD", "Cucumber"]
excerpt: >-
  Master logical execution filtering by using Gherkin Tags to dynamically orchestrate Smoke, Regression, and Sanity test suites.
readTime: 4 min read
---

As your enterprise test suite grows, you will inevitably accumulate hundreds of Gherkin scenarios. Running the entire suite for every minor code change is inefficient. This is where **Cucumber Tags** come to the rescue.

In this tutorial, we will explore how to use tags to create targeted, dynamic test executions without modifying a single line of code.

### What are Tags?

Tags are annotations starting with `@` that you place in your `.feature` files above a `Feature`, `Scenario`, or `Scenario Outline`. They act as logical groupings.

For example, in our `advanced-gherkin.feature` file:

```gherkin
@ecommerce @regression
Feature: E-Commerce Cart Management
 
  @smoke
  Scenario: Adding a single item to the cart
    When the user navigates to the "Electronics" category
    # ...
 
  @data-driven
  Scenario Outline: Applying promotional discount codes
    # ...
```

By default, running `npx cucumber-js` executes everything. But we can utilize the `--tags` CLI flag to filter exactly what runs.

### 1. Running a Specific Tag

If a developer makes a small fix and only wants to run the core smoke tests to ensure they didn't break anything critical, they can execute:

```bash
npx cucumber-js features/advanced-gherkin.feature --tags "@smoke"
```

**Execution Output:**

Notice how Cucumber ignores all the other scenarios in the file and exclusively targets the one tagged with `@smoke`!

```text
[Hook: BeforeAll] Bootstrapping Database Connections...
[Hook: BeforeAll] Launching Playwright Browser Context...
 
[Hook: Before] Starting Scenario: Adding a single item to the cart
[Setup] User authenticated successfully.
[Setup] Cart cleared.
Navigating to category: Electronics
Adding product to cart: Wireless Headphones
Asserting cart badge equals: 1
Asserting cart total is: $99.00
[Hook: After] Scenario Adding a single item to the cart executed successfully.
 
[Hook: AfterAll] Closing Browser Context...
[Hook: AfterAll] Tearing down Database Connections...
 
8 hooks (8 passed)
1 scenario (1 passed)
8 steps (8 passed)
0m 4.748s
```

### 2. Complex Tag Logic (AND / OR / NOT)

Cucumber supports boolean logic for advanced test filtering during pipeline execution:

**OR Execution:** Run scenarios that have *either* tag.

```bash
npx cucumber-js --tags "@smoke or @data-driven"
```

**AND Execution:** Run scenarios that have *both* tags.

```bash
npx cucumber-js --tags "@ecommerce and @smoke"
```

**NOT Execution:** Run everything *except* flaky tests.

```bash
npx cucumber-js --tags "not @flaky"
```

### Summary

Tags transform a monolithic test suite into a highly modular execution engine. By strategically tagging your Gherkin scenarios, CI/CD pipelines can dynamically trigger specific test suites (like `Sanity`, `Regression`, or `Smoke`) based on the context of the deployment!
