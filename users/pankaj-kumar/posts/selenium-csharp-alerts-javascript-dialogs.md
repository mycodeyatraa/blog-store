---
title: "Handling Alerts and Javascript Dialogs in Selenium C#"
date: "08-Jan-2025"
description: "Master Javascript Alerts, Confirms, and Prompts using Selenium C#. Learn how to use driver.SwitchTo().Alert() to read text, accept, dismiss, and send keys."
categories: ["Selenium C#", "Test Automation"]
tags: ["Selenium", "C#", ".NET", "Alerts", "SwitchTo", "Javascript Dialogs"]
author: "Pankaj Kumar"
lastUpdated: "08-Jan-2025"
---

Welcome back to the **Selenium C# Mastery Series**!

Have you ever clicked a button and a popup appeared at the top of the browser saying "Are you sure?" 

Those are **JavaScript Dialogs**. Unlike normal HTML elements, they are rendered by the browser itself. Because they are not part of the DOM, you cannot use `FindElement` to interact with them!

To handle them, Selenium provides a dedicated interface: **`IAlert`**.

---

### The Three Types of Alerts

JavaScript has three distinct types of dialogs:

1. **Simple Alert (`alert()`)**: Contains a message and a single **OK** button.
2. **Confirmation Alert (`confirm()`)**: Contains a message with **OK** and **Cancel** buttons.
3. **Prompt Alert (`prompt()`)**: Contains a message, a text input field, **OK**, and **Cancel**.

### Controlling Alerts with `IAlert`

To interact with a dialog, you must first switch Selenium's focus away from the main webpage and onto the alert itself:

```csharp
IAlert alert = driver.SwitchTo().Alert();
```

Once switched, you have four commands at your disposal:
* `alert.Accept()`: Clicks the "OK" button.
* `alert.Dismiss()`: Clicks the "Cancel" button.
* `alert.Text`: Reads the message displayed on the popup.
* `alert.SendKeys("text")`: Types text into a Prompt popup.

---

### Handling Alerts in C#

Let's write a script! We will create a new class called `Blog7_Alerts`. To guarantee we can test all three alert types, we will use an advanced Selenium technique: using `IJavaScriptExecutor` to force the browser to spawn the alerts for us!

```csharp
using System;
using OpenQA.Selenium;
using OpenQA.Selenium.Chrome;
using System.Threading;
namespace SeleniumCSharpTutorial
{
    public class Blog7_Alerts
    {
        public static void Run()
        {
            Console.WriteLine("Initializing ChromeDriver for Alerts...");
            ChromeOptions options = new ChromeOptions();
            options.AddArgument("--headless");
            options.AddArgument("--disable-gpu");
            options.AddArgument("--window-size=1920,1080");
            using (IWebDriver driver = new ChromeDriver(options))
            {
                driver.Navigate().GoToUrl("https://practice.mycodeyatra.com/#/form-practice");
                Console.WriteLine("Navigated to Form Practice Site.");
                Thread.Sleep(2000); 
                // We use IJavaScriptExecutor to forcefully spawn alerts!
                IJavaScriptExecutor js = (IJavaScriptExecutor)driver;
                Console.WriteLine("\n--- 1. HANDLING SIMPLE ALERTS ---");
                js.ExecuteScript("alert('Hello! I am a simple alert box!');");
                Thread.Sleep(1000); // Wait for alert to render
                IAlert simpleAlert = driver.SwitchTo().Alert();
                Console.WriteLine("Alert Text: " + simpleAlert.Text);
                simpleAlert.Accept(); // Clicks OK
                Console.WriteLine("Simple Alert Accepted!");
                Console.WriteLine("\n--- 2. HANDLING CONFIRMATION ALERTS ---");
                js.ExecuteScript("confirm('Do you want to proceed?');");
                Thread.Sleep(1000);
                IAlert confirmAlert = driver.SwitchTo().Alert();
                Console.WriteLine("Confirm Alert Text: " + confirmAlert.Text);
                confirmAlert.Dismiss(); // Clicks Cancel
                Console.WriteLine("Confirm Alert Dismissed (Cancelled)!");
                Console.WriteLine("\n--- 3. HANDLING PROMPT ALERTS ---");
                js.ExecuteScript("prompt('Please enter your name:', 'Harry Potter');");
                Thread.Sleep(1000);
                IAlert promptAlert = driver.SwitchTo().Alert();
                Console.WriteLine("Prompt Alert Text: " + promptAlert.Text);
                promptAlert.SendKeys("Pankaj Kumar"); // Types into the prompt
                promptAlert.Accept(); // Clicks OK
                Console.WriteLine("Prompt Alert text entered and Accepted!");
                Console.WriteLine("\nAlerts script executed successfully!");
            }
        }
    }
}
```

> **💡 Important Reminder:** If you try to switch to an alert that hasn't appeared yet, Selenium will throw a `NoAlertPresentException`. Always ensure the alert is fully rendered (using Waits) before calling `.SwitchTo().Alert()`!

---

### Executing the Code

Update your `Program.cs` to call `Blog7_Alerts.Run();` and execute the application:

```bash
dotnet run
```

### 🎯 Console Output

```text
Initializing ChromeDriver for Alerts...
Navigated to Form Practice Site.
 
--- 1. HANDLING SIMPLE ALERTS ---
Alert Text: Hello! I am a simple alert box!
Simple Alert Accepted!
 
--- 2. HANDLING CONFIRMATION ALERTS ---
Confirm Alert Text: Do you want to proceed?
Confirm Alert Dismissed (Cancelled)!
 
--- 3. HANDLING PROMPT ALERTS ---
Prompt Alert Text: Please enter your name:
Prompt Alert text entered and Accepted!
 
Alerts script executed successfully!
```

### Summary

You have officially mastered Browser Dialogs! You can now effortlessly accept standard alerts, dismiss confirmations, and pass data directly into Javascript Prompts.

In our next blog, we will tackle navigating complex web applications: **Switching Windows and Tabs**!

Keep coding, and see you next time!
