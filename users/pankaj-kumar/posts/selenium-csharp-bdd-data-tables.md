---
title: "Data-Driven BDD: Reqnroll Data Tables & Scenario Outlines"
date: "29-Jan-2025"
description: "Master data-driven testing in Behavior-Driven Development! Learn how to use Scenario Outlines and Data Tables to pass complex data into your Reqnroll C# steps."
categories: ["Selenium C#", "Test Automation", "BDD"]
tags: ["Selenium", "C#", ".NET", "BDD", "Reqnroll", "SpecFlow", "Data-Driven", "Framework Design"]
author: "Pankaj Kumar"
lastUpdated: "29-Jan-2025"
---

Welcome back to the **BDD Automation with Reqnroll** series!

We've successfully learned how to write basic Gherkin scenarios and bind them to our C# Selenium code using Context Injection. But what happens when you need to test the login page with 10 different combinations of usernames and passwords?

Writing 10 separate `Scenario:` blocks would be a massive waste of time and create unmaintainable Feature files. 

Today, we will learn how to achieve **Data-Driven Testing natively in BDD** using two powerful Reqnroll features: **Scenario Outlines** and **Data Tables**.

---

## 1. Scenario Outlines

A `Scenario Outline` allows you to write a test scenario *once* and execute it multiple times using a table of data provided in an `Examples:` block.

Let's update our `Login.feature` file to test multiple users!

```gherkin
Feature: User Authentication
  Scenario Outline: Data-Driven Login Test
    Given I navigate to the MyCodeYatra login page
    When I enter my username "<Email>"
    And I enter my password "<Password>"
    And I click the login button
    Then I should see the dashboard page
  Examples:
    | Email                       | Password      |
    | admin@mycodeyatra.com       | AdminPass123! |
    | standard@mycodeyatra.com    | UserPass123!  |
    | lockedout@mycodeyatra.com   | LockedPass!   |
```

### How does this work?
Notice that we replaced the hardcoded string with `<Email>` and `<Password>`. Reqnroll will dynamically read the `Examples` table below it, and automatically execute this exact scenario **three separate times**! Your C# Step Definitions (`LoginSteps.cs`) do not need to change at all! The strings are automatically injected via Regex!

---

## 2. Data Tables

A Scenario Outline executes the *entire scenario* multiple times. But what if you want to pass a large list of data into a *single step*? 

For example, what if you are filling out a massive User Registration form? You wouldn't want to write 15 different `When` steps for First Name, Last Name, Phone, Address, etc. 

Instead, you use a **Data Table**.

Create a new file named `Registration.feature`:

```gherkin
Feature: User Registration
  Scenario: Register a new user
    Given I navigate to the registration page
    When I submit the following user details:
      | FirstName | LastName | Email                  | Phone      |
      | Pankaj    | Kumar    | pankaj@mycodeyatra.com | 555-0199   |
    Then my account should be successfully created
```

---

## 3. Handling Data Tables in C#

To process that beautiful Markdown-style table in our C# code, Reqnroll provides an amazing helper method called `CreateInstance<T>()`. 

First, let's create a simple C# object to hold our data inside a new folder called `Models`:

```cs
// Models/UserModel.cs
namespace mcyt_sel_csharp.Models
{
    public class UserModel
    {
        public string FirstName { get; set; }
        public string LastName { get; set; }
        public string Email { get; set; }
        public string Phone { get; set; }
    }
}
```

Now, we write our Step Definition! Create `RegistrationSteps.cs`:

```cs
using Reqnroll;
using mcyt_sel_csharp.Models;
using System;
namespace mcyt_sel_csharp.StepDefinitions
{
    [Binding]
    public class RegistrationSteps
    {
        [When(@"I submit the following user details:")]
        public void WhenISubmitTheFollowingUserDetails(Table table)
        {
            // Magic happens here! Reqnroll maps the table directly to our C# Class!
            var user = table.CreateInstance<UserModel>();
            Console.WriteLine($"Filling out form for {user.FirstName} {user.LastName}");
            Console.WriteLine($"Email: {user.Email}");
            Console.WriteLine($"Phone: {user.Phone}");
            // In reality, you would pass this 'user' object into your Page Object Model!
            // _registrationPage.FillForm(user);
        }
    }
}
```

### The Magic of `CreateInstance<T>()`
Instead of manually writing loops to read rows and columns from the `Table` parameter, `CreateInstance<T>()` automatically looks at the header names in your feature file (`FirstName`, `LastName`) and perfectly maps them to the properties in your `UserModel.cs` class! 

*(Note: If you have multiple rows in your Data Table, you can use `table.CreateSet<UserModel>()` to return an `IEnumerable` list of objects!)*

---

## Conclusion

By mastering **Scenario Outlines** and **Data Tables**, your BDD feature files become incredibly concise and powerful. Product Managers can supply thousands of test data permutations directly into the plain English `.feature` files, and your C# code handles it dynamically without ever needing to be rewritten.

In our next blog, we will explore **Reqnroll Hooks & Tags**, allowing us to control exactly which scenarios run in parallel, and how to execute setup logic selectively!

Happy Automating!
