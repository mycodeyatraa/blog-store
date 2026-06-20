---
title: "The BaseTest Class: Abstracting Setup and Teardown in Selenium C#"
date: "22-Jan-2025"
description: "Eliminate boilerplate code from your test classes by building a robust BaseTest class that handles Browser Factory initialization and Teardown automatically."
categories: ["Selenium C#", "Test Automation"]
tags: ["Selenium", "C#", ".NET", "BaseTest", "NUnit", "Framework Design"]
author: "Pankaj Kumar"
lastUpdated: "22-Jan-2025"
---

Welcome back to the **Selenium C# Mastery Series**!

Take a look at any test class we have written so far. What do you see? In every single file, we are repeating the exact same `[SetUp]` and `[TearDown]` logic.

```cs
[SetUp]
public void Setup()
{
    BrowserType type = BrowserFactory.GetBrowserType(ConfigReader.Browser);
    driver = BrowserFactory.InitDriver(type);
    driver.Manage().Timeouts().ImplicitWait = TimeSpan.FromSeconds(ConfigReader.ImplicitWait);
}
[TearDown]
public void TearDown()
{
    if (driver != null)
    {
        driver.Quit();
        driver.Dispose();
    }
}
```

If you have 50 test classes and decide to add a logging mechanism to your setup, you would have to update 50 files! This violates the **DRY (Don't Repeat Yourself)** principle.

Today, we will solve this by introducing the **BaseTest Class**.

---

## 1. What is a BaseTest?

A BaseTest is a parent class that contains all the common configuration, initialization, and cleanup logic required by your test scripts. 

Because NUnit automatically executes `[SetUp]` and `[TearDown]` methods found in parent classes, any child class that inherits from the BaseTest gets this functionality completely for free!

---

## 2. Creating the BaseTest Class

Let's create a new file named `BaseTest.cs` inside a new `Tests` folder (or at your project root).

```cs
using NUnit.Framework;
using OpenQA.Selenium;
using mcyt_sel_csharp.Core;
using System;
namespace mcyt_sel_csharp.Tests
{
    public class BaseTest
    {
        // Protected so child classes can access the driver!
        protected IWebDriver driver;
        [SetUp]
        public void GlobalSetup()
        {
            Console.WriteLine("--- Executing BaseTest Setup ---");
            // 1. Read the browser type from our Global Configuration (appsettings.json)
            BrowserType type = BrowserFactory.GetBrowserType(ConfigReader.Browser);
            // 2. Initialize the WebDriver using the Factory
            driver = BrowserFactory.InitDriver(type);
            // 3. Apply global timeouts
            driver.Manage().Timeouts().ImplicitWait = TimeSpan.FromSeconds(ConfigReader.ImplicitWait);
        }
        [TearDown]
        public void GlobalTearDown()
        {
            Console.WriteLine("--- Executing BaseTest TearDown ---");
            if (driver != null)
            {
                driver.Quit();
                driver.Dispose();
            }
        }
    }
}
```

---

## 3. Refactoring Our Test Script

Now, let's create our test class. Look at how incredibly clean and focused it becomes! We no longer care about *how* the browser starts or closes. We only care about the test logic.

Create `Blog21_BaseTestImplementation.cs`:

```cs
using NUnit.Framework;
using mcyt_sel_csharp.Tests;
using mcyt_sel_csharp.Core;
namespace mcyt_sel_csharp
{
    [TestFixture]
    // Notice the inheritance: ' : BaseTest '
    public class Blog21_BaseTestImplementation : BaseTest
    {
        [Test]
        public void TestWithInheritedSetup()
        {
            // The 'driver' object is automatically available because it is 'protected' in BaseTest!
            driver.Navigate().GoToUrl(ConfigReader.BaseUrl);
            Assert.That(driver.Url, Does.Contain("practice"), "Failed to land on practice page!");
        }
    }
}
```

### The Magic of NUnit Inheritance
When you execute `TestWithInheritedSetup()`, NUnit does the following in order:
1. Executes `BaseTest.GlobalSetup()` -> Starts the browser.
2. Executes `Blog21_BaseTestImplementation.TestWithInheritedSetup()` -> Runs your test logic.
3. Executes `BaseTest.GlobalTearDown()` -> Closes the browser.

---

## 4. Running the Test

Run the test using the .NET CLI:

```sh
dotnet test --filter "Blog21"
```

**Output:**

```
--- Executing BaseTest Setup ---
--- Executing BaseTest TearDown ---
Passed!  - Failed:     0, Passed:     1, Skipped:     0, Total:     1
```

---

## Conclusion

By inheriting from a **BaseTest**, your actual test files shrink by over 50%. You have completely separated the **framework infrastructure** (starting drivers, reading configs) from your **business logic** (testing the application).

This is what a true Enterprise Automation Framework looks like!

In the next blog, we will tackle **Logging and Reporting**. We will integrate **ExtentReports** into our framework to generate beautiful, interactive HTML dashboards of our test results!

Happy Automating!
