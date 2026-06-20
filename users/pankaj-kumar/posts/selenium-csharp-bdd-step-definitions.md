---
title: "Reqnroll Step Definitions & Context Injection in Selenium C#"
date: "28-Jan-2025"
description: "Learn how to bridge the gap between English Gherkin files and C# Selenium code using Reqnroll Step Definitions and powerful Context Injection."
categories: ["Selenium C#", "Test Automation", "BDD"]
tags: ["Selenium", "C#", ".NET", "BDD", "Reqnroll", "SpecFlow", "Step Definitions", "Dependency Injection", "Framework Design"]
author: "Pankaj Kumar"
lastUpdated: "28-Jan-2025"
---

Welcome back to the **BDD Automation with Reqnroll** series!

In our previous blog, we wrote a `Login.feature` file containing plain English sentences. However, if we attempt to run it, the test will fail and say: **"No matching step definition found for the step"**.

Reqnroll has no idea how to "navigate to a page". It needs us to explicitly link those English sentences to our Selenium C# methods. Today, we will build that bridge using **Step Definitions** and explore **Context Injection**—the modern, preferred way to share WebDriver instances across BDD steps without using static variables!

---

## 1. Creating the Step Definitions

Create a new folder in your project named `StepDefinitions`. Inside it, create a class named `LoginSteps.cs`.

To tell Reqnroll that this class contains test code, we must decorate it with the `[Binding]` attribute.

```cs
using Reqnroll;
using mcyt_sel_csharp.Pages;
using NUnit.Framework;
using OpenQA.Selenium;
namespace mcyt_sel_csharp.StepDefinitions
{
    [Binding]
    public class LoginSteps
    {
        private readonly IWebDriver _driver;
        private readonly LoginPage _loginPage;
        // Where does this driver come from? We'll see in the next section!
        public LoginSteps(IWebDriver driver)
        {
            _driver = driver;
            _loginPage = new LoginPage(_driver);
        }
        [Given(@"I navigate to the MyCodeYatra login page")]
        public void GivenINavigateToTheMyCodeYatraLoginPage()
        {
            _driver.Navigate().GoToUrl("https://mycodeyatra.com/practice/login");
        }
        [When(@"I enter my username ""(.*)""")]
        public void WhenIEnterMyUsername(string username)
        {
            _loginPage.EnterUsername(username);
        }
        [When(@"I enter my password ""(.*)""")]
        public void WhenIEnterMyPassword(string password)
        {
            _loginPage.EnterPassword(password);
        }
        [When(@"I click the login button")]
        public void WhenIClickTheLoginButton()
        {
            _loginPage.ClickLogin();
        }
        [Then(@"I should see the dashboard page")]
        public void ThenIShouldSeeTheDashboardPage()
        {
            Assert.That(_driver.Url, Does.Contain("dashboard"));
        }
    }
}
```

### Understanding Step Definitions
- **[Binding]**: Marks the class so Reqnroll knows to scan it for steps.
- **[Given], [When], [Then]**: Attributes containing Regular Expressions (Regex) that exactly match the English text in the `.feature` file.
- **Regex Parameters**: Notice how `"(.*)"` automatically extracts the string `"admin@mycodeyatra.com"` from the feature file and passes it into our C# method as the `username` parameter!

---

## 2. Context Injection (Dependency Injection)

You might have noticed something strange in our constructor. We requested an `IWebDriver` object, but we never initialized it. How does Reqnroll know what `_driver` is?

In traditional NUnit tests, we inherited from a `BaseTest` class to get our driver. But in Reqnroll (and its predecessor SpecFlow), inheriting base classes containing state is considered **bad practice** because it breaks parallel execution.

Instead, Reqnroll uses **Context Injection** (a lightweight IoC container). If we want an `IWebDriver`, we simply ask for it in the constructor. Reqnroll will look inside its internal container, grab the driver, and hand it to us!

But first, we must put the driver *into* the container. We do this using **Reqnroll Hooks**.

Create a new file in your project named `Hooks.cs`:

```cs
using Reqnroll;
using OpenQA.Selenium;
using OpenQA.Selenium.Chrome;
using Reqnroll.BoDi; // This is Reqnroll's Dependency Injection library!
namespace mcyt_sel_csharp.Support
{
    [Binding]
    public class Hooks
    {
        private readonly IObjectContainer _container;
        // Reqnroll injects its own Object Container into this Hook!
        public Hooks(IObjectContainer container)
        {
            _container = container;
        }
        [BeforeScenario]
        public void SetupWebDriver()
        {
            // 1. Initialize our WebDriver (you can use WebDriverFactory here!)
            IWebDriver driver = new ChromeDriver();
            driver.Manage().Window.Maximize();
            // 2. Register it with Reqnroll's dependency container!
            _container.RegisterInstanceAs<IWebDriver>(driver);
        }
        [AfterScenario]
        public void TearDownWebDriver()
        {
            // 1. Ask the container for the driver
            var driver = _container.Resolve<IWebDriver>();
            // 2. Quit the browser
            driver?.Quit();
            driver?.Dispose();
        }
    }
}
```

### Why Context Injection is Powerful
Imagine you have `LoginSteps.cs` and `CheckoutSteps.cs`. If both classes need the WebDriver, you just add `IWebDriver` to both constructors. Reqnroll guarantees they both receive the exact same, shared Chrome instance for that specific scenario! When the scenario ends, `[AfterScenario]` cleanly disposes of it.

---

## Conclusion

We have successfully bridged the gap! Our plain English feature files are now deeply connected to our C# Selenium logic through **Step Definitions**. We also architected a highly scalable, thread-safe framework using **Reqnroll's Context Injection**, entirely avoiding static variables and messy inheritance.

In our next blog, we will scale our tests even further by passing complex **Data Tables** and executing massive **Scenario Outlines**!

Happy Automating!
