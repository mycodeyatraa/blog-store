---
title: Speaking the Language: A Guide to Gherkin Syntax
date: 09-Jan-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, bdd, cucumber, gherkin, given-when-then]
category: Selenium TypeScript
categories: [Selenium TypeScript, BDD & Cucumber Framework]
excerpt: >-
  Master the Given, When, Then grammar of Gherkin syntax. Learn how to write structured, declarative scenarios and backgrounds that Product Managers can read and Cucumber can execute.
readTime: 4 min read
---

# Speaking the Language: A Guide to Gherkin Syntax

In our introduction to Behavior-Driven Development (BDD), we looked at a simple plain-English file that described a login process. That plain-English file was written in a strict grammar called **Gherkin**.

Gherkin is a line-oriented language that uses indentation to define structure. It is designed to be readable by non-technical stakeholders (like Product Managers) while being structured enough that automation frameworks (like Cucumber) can parse it into executable TypeScript code.

In this tutorial, we will explore the core keywords that make up the Gherkin syntax.

---

## 1. The Core Keywords

Every Gherkin step must begin with a keyword. Let's break them down:

1. **Feature**: The highest level description of the software feature being tested.
2. **Scenario**: A specific, single test case representing one path through the feature.
3. **Given**: Establishes the initial state or preconditions before an action occurs.
4. **When**: Describes the primary action or event triggered by the user.
5. **Then**: Asserts the expected outcome or observable result.
6. **And / But**: Used to link multiple `Given`, `When`, or `Then` statements together without repeating the keyword.
7. **Background**: A special block of steps that run *before every single Scenario* in the file (perfect for logging in!).

---

## 2. Writing a Comprehensive Feature File

Let's look at how these keywords interact. Create a file at `tests/bdd/features/syntax_demo.feature`:

```gherkin
Feature: E-Commerce Shopping Cart Validation
  In order to purchase items
  As a registered customer
  I want to be able to add products to my cart and check out
  Background: 
    Given the user is on the "MyCodeYatra Store" homepage
    And the user is logged in with valid credentials
  Scenario: Adding a single item to the cart
    When the user searches for "Selenium Masterclass Book"
    And the user clicks the "Add to Cart" button
    Then the cart badge should display "1"
    And the "Selenium Masterclass Book" should be visible in the cart summary
    But the "Checkout" button should remain disabled until terms are accepted
  Scenario: Attempting to checkout without accepting terms
    Given the cart contains 1 item
    When the user navigates to the "Checkout" page
    Then an error message "Please accept the Terms and Conditions" should be displayed
```

### Breaking it down:
- The `Background` runs first. For *both* scenarios, the browser will open and the user will log in. This prevents us from having to write "Given the user logs in" inside every single scenario!
- We use `And` extensively to make the sentences read naturally.
- We use `But` to assert a negative or contrasting condition (e.g., the button should *not* be enabled).

## 3. Best Practices for Gherkin

When writing Gherkin, adhere to these rules to prevent your files from becoming unmaintainable messes:

1. **Keep it Declarative, not Imperative:** Don't write "Given the user clicks the xpath //input[@id='email'] and types admin". Write "Given the user logs in". Hide the technical details!
2. **One Action per Scenario:** A `Scenario` should test exactly one behavior. Don't write a mega-scenario that logs in, buys an item, checks the profile, and logs out. 
3. **Write for the Business:** If a Product Manager reads your file and doesn't understand it, your Gherkin is flawed.

## Conclusion

Gherkin bridges the gap between the business and the engineering team by providing a shared, unambiguous vocabulary.

However, writing English sentences doesn't magically automate a browser. The Cucumber engine needs to know *what code to run* when it reads "Given the user logs in".

In our next tutorials, we will install the **Cucumber** engine and write **Step Definitions**—the TypeScript glue code that binds our Gherkin sentences to our Selenium Webdriver commands!
