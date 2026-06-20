---
title: "Cross-Browser Testing: Building a WebDriver Factory in Selenium C#"
date: "20-Jan-2025"
description: "Stop hardcoding ChromeDriver! Learn how to implement the Factory Design Pattern to dynamically launch Chrome, Firefox, or Edge based on configuration."
categories: ["Selenium C#", "Test Automation"]
tags: ["Selenium", "C#", ".NET", "WebDriver Factory", "Cross Browser Testing", "Design Patterns", "Framework Design"]
author: "Pankaj Kumar"
lastUpdated: "20-Jan-2025"
---

Welcome back to the **Selenium C# Mastery Series**!

Up until now, in every single test we've written, our `[SetUp]` method looks exactly like this:

```cs
[SetUp]
public void Setup()
{
    driver = new ChromeDriver(); // Hardcoded!
    driver.Manage().Window.Maximize();
}
```

This works great for learning, but what happens when your manager asks you to run the entire test suite on **Microsoft Edge** or **Mozilla Firefox**? You would have to manually edit the code across all your test classes. This is a massive maintenance nightmare.

Today, we will solve this by implementing a core software engineering concept: **The Factory Design Pattern**.

---

## 1. What is a WebDriver Factory?

The Factory Pattern is a creational design pattern that provides an interface for creating objects in a superclass, but allows subclasses (or configuration) to alter the type of objects that will be created.

In automation, a **WebDriver Factory** is simply a dedicated class whose *only* job is to evaluate a string (like `"Chrome"`, `"Firefox"`, or `"Edge"`) and return the corresponding `IWebDriver` instance. 

---

## 2. Creating the Factory Class

Let's create a new folder in our project called `Core` (or `Utilities`) and add a `BrowserFactory.cs` file.

To make our implementation type-safe, we will use a C# `enum` to define the supported browsers.

```cs
using OpenQA.Selenium;
using OpenQA.Selenium.Chrome;
using OpenQA.Selenium.Edge;
using OpenQA.Selenium.Firefox;
using System;
namespace mcyt_sel_csharp.Core
{
    public enum BrowserType
    {
        Chrome,
        Firefox,
        Edge
    }
    public class BrowserFactory
    {
        public static IWebDriver InitDriver(BrowserType browserType)
        {
            IWebDriver driver;
            switch (browserType)
            {
                case BrowserType.Chrome:
                    ChromeOptions chromeOptions = new ChromeOptions();
                    // You can add headless mode, incognito, etc. here
                    driver = new ChromeDriver(chromeOptions);
                    break;
                case BrowserType.Firefox:
                    FirefoxOptions firefoxOptions = new FirefoxOptions();
                    driver = new FirefoxDriver(firefoxOptions);
                    break;
                case BrowserType.Edge:
                    EdgeOptions edgeOptions = new EdgeOptions();
                    driver = new EdgeDriver(edgeOptions);
                    break;
                default:
                    throw new ArgumentException($"Browser {browserType} is not supported!");
            }
            driver.Manage().Window.Maximize();
            return driver;
        }
    }
}
```

### Why `public static`?
Making the `InitDriver` method static allows us to call `BrowserFactory.InitDriver()` without having to create an instance of the factory class in our tests.

---

## 3. Refactoring Our Tests

Now, let's create a new test file named `Blog19_WebDriverFactory.cs` and see how much cleaner our setup becomes!

```cs
using NUnit.Framework;
using OpenQA.Selenium;
using mcyt_sel_csharp.Core;
using System;
namespace mcyt_sel_csharp
{
    [TestFixture]
    public class Blog19_WebDriverFactory
    {
        private IWebDriver driver;
        [SetUp]
        public void Setup()
        {
            // We just ask the Factory for the driver we want!
            // In a real framework, this value would come from a JSON/XML config file or CI/CD pipeline argument.
            driver = BrowserFactory.InitDriver(BrowserType.Edge); 
            driver.Manage().Timeouts().ImplicitWait = TimeSpan.FromSeconds(10);
        }
        [Test]
        public void TestLaunchOnEdge()
        {
            Console.WriteLine("Navigating to MyCodeYatra using Edge...");
            driver.Navigate().GoToUrl("https://practice.mycodeyatra.com/");
            Assert.That(driver.Title, Does.Contain("MyCodeYatra"), "Title verification failed.");
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
    }
}
```

---

## 4. The Magic of the Factory

By centralizing the instantiation of our WebDrivers:
1. **Zero Duplication**: We only define `ChromeOptions` or window maximization logic in exactly one place.
2. **Instant Cross-Browser Compatibility**: Changing the entire test suite to run on Firefox is as simple as passing `BrowserType.Firefox`.
3. **CI/CD Readiness**: In the future, we can read an environment variable (e.g., `Environment.GetEnvironmentVariable("BROWSER")`) and pass it to the factory dynamically!

In our next blog, we will take exactly this concept and implement a **Global Configuration Manager** to read our browser type and base URLs from an external `appsettings.json` file!

Happy Automating!
