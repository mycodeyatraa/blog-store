---
title: Data-Driven Testing using Cucumber Scenario Outlines
date: 24-Apr-2025
lastUpdated: 24-Apr-2025
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: ["playwright", "typescript", "bdd", "cucumber", "data-driven"]
category: UI Automation
categories: ["UI Automation", "Playwright", "TypeScript", "BDD", "Cucumber"]
excerpt: >-
  Scale your automation test coverage exponentially by utilizing Gherkin Examples tables and Scenario Outlines for Data-Driven Testing.
readTime: 4 min read
---

Writing individual scenarios for every possible combination of test data is the fastest way to create a bloated, unmaintainable test suite. 

Instead of copying and pasting the exact same test steps just to change a username or a promo code, Cucumber offers **Data-Driven Testing (DDT)** natively through **Scenario Outlines** and **Examples** tables.

### What is a Scenario Outline?

A `Scenario Outline` allows you to write a test scenario once, using `<placeholder>` variables. You then provide a table of data under an `Examples` section. Cucumber will loop through the table and execute the scenario from scratch for *every single row*!

Let's look at a real-world Playwright implementation:

**File:** `features/advanced-gherkin.feature`

```gherkin
@data-driven
Scenario Outline: Applying promotional discount codes
  Given the user has added a "Laptop" priced at "$1000" to their cart
  When the user applies the promo code "<PromoCode>"
  Then the final cart total should be "<FinalPrice>"
  And a discount message "<Message>" should be displayed

  Examples:
    | PromoCode     | FinalPrice | Message               |
    | SAVE10        | $900       | 10% discount applied! |
    | BLACKFRIDAY   | $800       | 20% discount applied! |
    | INVALID123    | $1000      | Promo code invalid    |
```

Notice how we only wrote the 4 testing steps once, but we defined 3 entirely different test combinations in the Examples table (Valid Code, High-Value Code, and Invalid Code).

### Dynamic Execution

When you run this feature file, Cucumber will unroll the `Examples` table. It dynamically replaces the `<PromoCode>`, `<FinalPrice>`, and `<Message>` tags with the data from the row, and then triggers the Playwright automation natively!

```bash
npx cucumber-js features/advanced-gherkin.feature --name "Applying promotional discount codes"
```

**Execution Output:**

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

[Hook: Before] Starting Scenario: Applying promotional discount codes
[Tagged Hook: @data-driven] Injecting test data into scenario context...
[Setup] User authenticated successfully.
[Setup] Cart cleared.
Added Laptop at $1000
Applying Promo Code: BLACKFRIDAY
Asserting final price is $800
Asserting discount message: 20% discount applied!
[Hook: After] Scenario Applying promotional discount codes executed successfully.

...

3 scenarios (3 passed)
27 steps (27 passed)
```

As you can see, the single `Scenario Outline` was executed exactly 3 times in complete isolation.

### Summary

Scenario Outlines are the ultimate tool for scaling your automation coverage. By separating your business logic from your test data, you can achieve exponential testing coverage with minimal code maintenance!
