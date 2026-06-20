---
title: "Controlling Execution: Reqnroll Hooks & Tags in Selenium C#"
date: "30-Jan-2025"
description: "Learn how to orchestrate complex BDD test suites by categorizing tests with Tags and executing precise setup logic using Reqnroll Hooks."
categories: ["Selenium C#", "Test Automation", "BDD"]
tags: ["Selenium", "C#", ".NET", "BDD", "Reqnroll", "SpecFlow", "Hooks", "Tags", "Framework Design"]
author: "Pankaj Kumar"
lastUpdated: "30-Jan-2025"
---

Welcome to the grand finale of the **BDD Automation with Reqnroll** series!

So far, we have built a beautiful, data-driven, Dependency Injected BDD framework. However, as your enterprise framework scales to thousands of tests, two problems will emerge:

1. **Selective Execution**: How do you run only the "Smoke" tests before a quick deployment without running the entire 4-hour suite?
2. **Conditional Setup**: How do you open a Database Connection only for tests that actually need it, without slowing down the rest?

Today, we solve both problems simultaneously using **Reqnroll Tags** and **Scoped Hooks**.

---

## 1. What are Reqnroll Tags?

Tags are simple labels (starting with an `@` symbol) that you can attach to an entire `Feature` or to individual `Scenario` blocks. They serve as metadata for categorization.

Let's modify our `Login.feature` file to include tags:

```gherkin
@Authentication @Regression
Feature: User Authentication
  @Smoke @Priority1
  Scenario Outline: Successful Login with Valid Credentials
    Given I navigate to the MyCodeYatra login page
    When I enter my username "<Email>"
    And I enter my password "<Password>"
    And I click the login button
    Then I should see the dashboard page
  Examples:
    | Email                       | Password      |
    | admin@mycodeyatra.com       | AdminPass123! |
```

In the example above:
- *All* scenarios in this file inherit `@Authentication` and `@Regression`.
- The `Successful Login` scenario *additionally* possesses the `@Smoke` and `@Priority1` tags.

---

## 2. Running Tests by Tag

Because Reqnroll uses NUnit under the hood, these `@Tags` are automatically converted into NUnit `TestCategory` traits!

This means you can filter exactly which tests run using the .NET CLI:

**Run only Smoke Tests:**

```sh
dotnet test --filter TestCategory=Smoke
```

**Run all Authentication tests EXCEPT Smoke tests:**

```sh
dotnet test --filter "TestCategory=Authentication&TestCategory!=Smoke"
```

This is incredibly powerful when integrating with the **GitHub Actions CI/CD** pipeline we built earlier. You can configure GitHub to run only `@Smoke` tests when a developer opens a Pull Request, and the full `@Regression` suite when they merge to `main`!

---

## 3. Scoped Hooks (Conditional Setup)

In an earlier blog, we used `[BeforeScenario]` in our `Hooks.cs` file to inject the `IWebDriver`. This ran before *every single scenario*. 

But what if a specific test requires us to seed data into a Database before it runs? We don't want to slow down our UI tests by opening database connections unnecessarily!

Reqnroll allows you to **Scope** your hooks to specific tags:

```cs
using Reqnroll;
using System;
namespace mcyt_sel_csharp.Support
{
    [Binding]
    public class DatabaseHooks
    {
        // This Hook will ONLY execute for scenarios tagged with @DatabaseRequired!
        [BeforeScenario("DatabaseRequired")]
        public void SetupDatabaseConnection()
        {
            Console.WriteLine("--> [HOOK] Opening Heavy Database Connection...");
            // Execute SQL to setup test data
        }
        [AfterScenario("DatabaseRequired")]
        public void TeardownDatabaseConnection()
        {
            Console.WriteLine("--> [HOOK] Closing Database Connection...");
            // Clean up data
        }
        // This Hook runs for ALL scenarios
        [BeforeScenario(Order = 1)]
        public void GlobalSetup()
        {
            Console.WriteLine("--> [HOOK] Standard WebDriver Initialization...");
        }
    }
}
```

### Hook Ordering
Notice the `Order = 1` property? If multiple `[BeforeScenario]` hooks execute on the same test, Reqnroll executes them sequentially based on their `Order` (lowest number runs first).

For `[AfterScenario]`, the order is automatically **reversed** (highest number runs first), ensuring a perfect "First In, Last Out" teardown mechanism!

---

## Conclusion

By mastering **Tags** and **Scoped Hooks**, you now have ultimate control over the flow and execution of your test framework. 

You have successfully learned how to architect a fully-featured, deeply integrated BDD automation framework in C# using Reqnroll. You replaced hard-to-read NUnit files with business-friendly Gherkin specifications, and maintained clean architecture via Context Injection!

In our next and final series, we will break into the most advanced domain of Test Automation: **AI-Powered Testing using Semantic Kernel & LLMs**! 

Happy Automating!
