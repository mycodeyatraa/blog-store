---
title: "Advanced Interactions using the Actions Class in Selenium C#"
date: "11-Jan-2025"
description: "Master complex interactions in Selenium C# using the Actions class. Learn how to perform Mouse Hovers, Double Clicks, Right Clicks, and Keyboard Chaining."
categories: ["Selenium C#", "Test Automation"]
tags: ["Selenium", "C#", ".NET", "Actions", "MouseHover", "RightClick", "Keyboard"]
author: "Pankaj Kumar"
lastUpdated: "11-Jan-2025"
---

Welcome back to the **Selenium C# Mastery Series**!

We have already covered basic interactions like `.Click()` and `.SendKeys()`. But what happens when you need to automate complex user behaviors? How do you hover over a mega-menu to reveal dropdown links? How do you double-click a file, or right-click an image?

Enter the **`Actions`** class—Selenium's dedicated API for advanced Mouse and Keyboard interactions!

---

### The Anatomy of the Actions Class

The `Actions` class resides in the `OpenQA.Selenium.Interactions` namespace. 

When using `Actions`, you define a sequence of behaviors (chaining) and then trigger them all at once using the `.Perform()` method. If you forget to add `.Perform()` at the end of your chain, **nothing will happen**!

### Core Mouse & Keyboard Methods

Here are the most common methods you will use:
1. **Hovering**: `MoveToElement(element)`
2. **Double Click**: `DoubleClick(element)`
3. **Right Click**: `ContextClick(element)`
4. **Drag and Drop**: `DragAndDrop(source, target)`
5. **Keyboard Presses**: `KeyDown(Keys.Shift)` and `KeyUp(Keys.Shift)`

---

### Writing the C# Script

Let's put this into practice! We will create a new class called `Blog10_Actions`. In this script, we will hover over a button, double-click another button, right-click a header, and finally use a Keyboard Chain to hold `SHIFT` while typing a name!

```csharp
using System;
using OpenQA.Selenium;
using OpenQA.Selenium.Chrome;
using OpenQA.Selenium.Interactions;
using System.Threading;
namespace SeleniumCSharpTutorial
{
    public class Blog10_Actions
    {
        public static void Run()
        {
            Console.WriteLine("Initializing ChromeDriver for Actions Class...");
            ChromeOptions options = new ChromeOptions();
            options.AddArgument("--headless");
            options.AddArgument("--disable-gpu");
            options.AddArgument("--window-size=1920,1080");
            using (IWebDriver driver = new ChromeDriver(options))
            {
                driver.Navigate().GoToUrl("https://practice.mycodeyatra.com/#/form-practice");
                Console.WriteLine("Navigated to Main Page.");
                Thread.Sleep(2000); 
                // Instantiate the Actions class
                Actions actions = new Actions(driver);
                Console.WriteLine("\n--- 1. MOUSE HOVER (MoveToElement) ---");
                IWebElement submitBtn = driver.FindElement(By.XPath("//button[text()='Submit Form']"));
                // Perform a hover over the Submit button
                actions.MoveToElement(submitBtn).Perform();
                Console.WriteLine("Successfully hovered over the Submit Button.");
                Console.WriteLine("\n--- 2. DOUBLE CLICK ---");
                IWebElement clearBtn = driver.FindElement(By.XPath("//button[text()='Clear']"));
                actions.DoubleClick(clearBtn).Perform();
                Console.WriteLine("Successfully double-clicked the Clear Button.");
                Console.WriteLine("\n--- 3. RIGHT CLICK (ContextClick) ---");
                IWebElement header = driver.FindElement(By.TagName("h2"));
                actions.ContextClick(header).Perform();
                Console.WriteLine("Successfully right-clicked the Form Header.");
                Console.WriteLine("\n--- 4. KEYBOARD ACTIONS ---");
                IWebElement fullNameInput = driver.FindElement(By.Name("fullName"));
                // Click into the input, hold SHIFT, type text (it will be uppercase), release SHIFT
                actions.Click(fullNameInput)
                       .KeyDown(Keys.Shift)
                       .SendKeys("actions class rocks")
                       .KeyUp(Keys.Shift)
                       .Perform();
                Console.WriteLine("Typed text using Keyboard Shift: " + fullNameInput.GetAttribute("value"));
                Console.WriteLine("\nActions script executed successfully!");
            }
        }
    }
}
```

> **💡 Action Chaining:** Notice in the Keyboard section how we strung multiple methods together: `.Click().KeyDown().SendKeys().KeyUp().Perform()`. This builds a composite action sequence that Selenium executes rapidly in order!

---

### Executing the Code

Update your `Program.cs` to call `Blog10_Actions.Run();` and execute the application:

```bash
dotnet run
```

### 🎯 Console Output

```text
Initializing ChromeDriver for Actions Class...
Navigated to Main Page.

--- 1. MOUSE HOVER (MoveToElement) ---
Successfully hovered over the Submit Button.

--- 2. DOUBLE CLICK ---
Successfully double-clicked the Clear Button.

--- 3. RIGHT CLICK (ContextClick) ---
Successfully right-clicked the Form Header.

--- 4. KEYBOARD ACTIONS ---
Typed text using Keyboard Shift: ACTIONS CLASS ROCKS

Actions script executed successfully!
```

### Summary

You now have complete mastery over the user's physical hardware! The `Actions` class bridges the gap between simple test scripts and hyper-realistic user simulations.

This brings our **Selenium C# Mastery Series** to a phenomenal conclusion. You have successfully journeyed from setting up a basic C# Console project to orchestrating complex UI events across tabs, iframes, and dropdowns.

Happy automating, and thank you for reading!
