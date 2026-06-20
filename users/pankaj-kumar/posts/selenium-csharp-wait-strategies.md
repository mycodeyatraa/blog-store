---
title: "Wait Strategies in Selenium C#: Implicit and Explicit Waits"
date: "07-Jan-2025"
description: "Master Selenium C# synchronization. Learn the vital differences between Implicit Wait and Explicit Wait (WebDriverWait) to build stable, flake-free automation scripts."
categories: ["Selenium C#", "Test Automation"]
tags: ["Selenium", "C#", ".NET", "Waits", "WebDriverWait", "Implicit Wait", "Explicit Wait"]
author: "Pankaj Kumar"
lastUpdated: "07-Jan-2025"
---

Welcome back to the **Selenium C# Mastery Series**!

One of the biggest challenges in UI Automation is dealing with elements that haven't loaded yet. In modern web applications (like React, Angular, and Vue), elements appear and disappear asynchronously. If your Selenium script tries to interact with a button before the browser renders it, you'll be hit with the dreaded `NoSuchElementException`.

To solve this, we use **Wait Strategies**. In this blog, we will explore the two most critical types: **Implicit Waits** and **Explicit Waits**.

---

### 1. Implicit Wait

The Implicit Wait tells WebDriver to poll the DOM for a certain amount of time when trying to find *any* element if it is not immediately available.

**Pros:** You only define it once. It applies globally to all `FindElement` calls.
**Cons:** It only checks if the element is *present* in the DOM. It does *not* check if the element is visible, clickable, or enabled.

```csharp
driver.Manage().Timeouts().ImplicitWait = TimeSpan.FromSeconds(10);
```

### 2. Explicit Wait (WebDriverWait)

Explicit Wait allows your code to halt execution until a *specific condition* is met on a *specific element*. This is extremely powerful because you can wait for an element to become clickable, visible, or for an alert to be present.

In modern C# Selenium, we use `WebDriverWait` combined with lambda expressions.

```csharp
WebDriverWait wait = new WebDriverWait(driver, TimeSpan.FromSeconds(15));
IWebElement element = wait.Until(d => d.FindElement(By.Id("myId")));
```

### Writing the Code

Let's write a practical C# script to demonstrate both! To keep our repository organized, we will create a new class called `Blog6_Waits`.

```csharp
using System;
using OpenQA.Selenium;
using OpenQA.Selenium.Chrome;
using OpenQA.Selenium.Support.UI;
namespace SeleniumCSharpTutorial
{
    public class Blog6_Waits
    {
        public static void Run()
        {
            Console.WriteLine("Initializing ChromeDriver for Waits...");
            ChromeOptions options = new ChromeOptions();
            options.AddArgument("--headless");
            options.AddArgument("--disable-gpu");
            options.AddArgument("--window-size=1920,1080");
            using (IWebDriver driver = new ChromeDriver(options))
            {
                // 1. Implicit Wait
                driver.Manage().Timeouts().ImplicitWait = TimeSpan.FromSeconds(10);
                Console.WriteLine("Implicit Wait set to 10 seconds.");
                driver.Navigate().GoToUrl("https://practice.mycodeyatra.com/#/form-practice");
                Console.WriteLine("Navigated to Form Practice Site.");
                // Because of Implicit Wait, Selenium will keep trying for up to 10 seconds.
                IWebElement nameInput = driver.FindElement(By.Name("fullName"));
                nameInput.SendKeys("Testing Waits");
                Console.WriteLine("Found 'fullName' element using Implicit Wait!");
                // 2. Explicit Wait
                Console.WriteLine("Setting up Explicit Wait...");
                WebDriverWait wait = new WebDriverWait(driver, TimeSpan.FromSeconds(15));
                // Use a lambda expression to wait until the element is displayed
                IWebElement clearBtn = wait.Until(d => 
                {
                    IWebElement element = d.FindElement(By.XPath("//button[text()='Clear']"));
                    if (element.Displayed)
                    {
                        return element;
                    }
                    return null;
                });
                Console.WriteLine("Explicit Wait successfully found the 'Clear' button!");
                clearBtn.Click();
                Console.WriteLine("Waits script executed successfully!");
            }
        }
    }
}
```

> **💡 Best Practice Warning:** Never mix Implicit and Explicit waits on the same element! Doing so can cause unpredictable timeout durations where Selenium adds both times together.

---

### Executing the Code

Update your `Program.cs` to call `Blog6_Waits.Run();` and execute the application:

```bash
dotnet run
```

### 🎯 Console Output

```text
Initializing ChromeDriver for Waits...
Implicit Wait set to 10 seconds.
Navigated to Form Practice Site.
Found 'fullName' element using Implicit Wait!
Setting up Explicit Wait...
Explicit Wait successfully found the 'Clear' button!
Waits script executed successfully!
```

### Summary

Congratulations! You now have the tools to build completely stable and flake-free automation scripts. Use Implicit Wait as a baseline safety net, and deploy Explicit Waits for specific, tricky asynchronous elements.

In the next blog, we will learn how to handle annoying popups: **Alerts & Javascript Dialogs**!

Keep coding, and see you next time!
