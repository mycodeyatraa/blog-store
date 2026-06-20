---
title: "Basic Interactions in Selenium C#: Click, SendKeys, and Clear"
date: "04-Jan-2025"
description: "Learn the core interactions in Selenium C#. Master the use of .Click(), .SendKeys(), and .Clear() to interact with web forms, buttons, and text fields."
categories: ["Selenium C#", "Test Automation"]
tags: ["Selenium", "C#", ".NET", "SendKeys", "Click", "Clear", "WebDriver"]
author: "Pankaj Kumar"
lastUpdated: "04-Jan-2025"
---

Welcome back to the **Selenium C# Mastery Series**!

In the previous blog, we learned how to find web elements using locators like `By.Id` and `By.XPath`. However, finding an element is only half the battle. To simulate real user behavior, we must interact with those elements.

In this tutorial, we will explore the three most fundamental interactions in Selenium: **Typing text, clearing text, and clicking buttons.**

---

### The Big Three Interactions

When you interact with an `IWebElement` in C#, you will almost always use one of these three methods:

1. **`SendKeys(string text)`**: Used to type characters into an input field or text area.
2. **`Clear()`**: Used to erase any existing text inside an input field.
3. **`Click()`**: Used to click on buttons, links, checkboxes, or radio buttons.

Additionally, we often need to read what is currently inside a text box. For this, we use **`GetAttribute("value")`**.

### Writing the Interactions Script

Let's put this into practice! We will navigate to the MyCodeYatra Form Practice Sandbox. Our script will type a name, clear it, type a new name, fill out a bio, and finally click the "Clear" button to reset the entire form.

Open your `Program.cs` file and update it with the following code:

```csharp
using System;
using OpenQA.Selenium;
using OpenQA.Selenium.Chrome;
using System.Threading;
namespace SeleniumCSharpTutorial
{
    class Program
    {
        static void Main(string[] args)
        {
            Console.WriteLine("Initializing ChromeDriver for Interactions...");
            ChromeOptions options = new ChromeOptions();
            options.AddArgument("--headless");
            options.AddArgument("--disable-gpu");
            options.AddArgument("--window-size=1920,1080");
            using (IWebDriver driver = new ChromeDriver(options))
            {
                driver.Navigate().GoToUrl("https://practice.mycodeyatra.com/#/form-practice");
                Console.WriteLine("Navigated to Form Practice Site.");
                Thread.Sleep(2000); // Allow React to mount
                // 1. Locate the elements
                IWebElement fullNameInput = driver.FindElement(By.Name("fullName"));
                IWebElement bioInput = driver.FindElement(By.Name("bio"));
                IWebElement clearFormBtn = driver.FindElement(By.XPath("//button[text()='Clear']"));
                // 2. Interaction: SendKeys (Typing text)
                Console.WriteLine("Typing text into the Full Name field...");
                fullNameInput.SendKeys("John Doe");
                // 3. Getting text from an input using GetAttribute("value")
                var enteredName = fullNameInput.GetAttribute("value");
                Console.WriteLine("Text currently in field: " + enteredName);
                // 4. Interaction: Clear (Removing text)
                Console.WriteLine("Clearing the text field...");
                fullNameInput.Clear();
                Console.WriteLine("Entering the correct name and bio...");
                fullNameInput.SendKeys("Pankaj Kumar");
                bioInput.SendKeys("Automation Architect and SDET");
                // 5. Interaction: Click (Clicking a button)
                Console.WriteLine("Clicking the 'Clear Form' button to reset the entire form...");
                clearFormBtn.Click();
                var finalNameValue = fullNameInput.GetAttribute("value");
                Console.WriteLine("Text in Full Name after clicking Clear button: '" + finalNameValue + "'");
                Console.WriteLine("Interactions script executed successfully!");
            }
        }
    }
}
```

> **💡 Pro Tip (`GetAttribute` vs `Text`):** You might be tempted to use `.Text` to read what you typed into an input field. However, `.Text` only reads the *inner HTML text* between tags (like `<p>text</p>`). To read the value of an `<input>` tag, you must use `.GetAttribute("value")`!

---

### Executing the Code

Run your C# application using the terminal:

```bash
dotnet run
```

### 🎯 Console Output

```text
Initializing ChromeDriver for Interactions...
Navigated to Form Practice Site.
Typing text into the Full Name field...
Text currently in field: John Doe
Clearing the text field...
Entering the correct name and bio...
Clicking the 'Clear Form' button to reset the entire form...
Text in Full Name after clicking Clear button: ''
Interactions script executed successfully!
```

### Summary

You have successfully mastered the art of basic interactions! You can now type text with `SendKeys()`, delete text with `Clear()`, read values with `GetAttribute("value")`, and trigger actions with `Click()`.

In the next blog, we will tackle one of the most common challenges in UI automation: **Handling Dropdowns using the `SelectElement` class!**

Stay tuned, and happy coding!
