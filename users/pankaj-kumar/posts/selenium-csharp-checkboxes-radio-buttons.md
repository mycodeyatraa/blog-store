---
title: "Handling Checkboxes and Radio Buttons in Selenium C#"
date: "06-Jan-2025"
description: "Learn how to expertly handle checkboxes and radio buttons in Selenium C#. Understand how to use the .Selected property to verify state before clicking."
categories: ["Selenium C#", "Test Automation"]
tags: ["Selenium", "C#", ".NET", "Checkboxes", "Radio Buttons", "WebDriver"]
author: "Pankaj Kumar"
lastUpdated: "06-Jan-2025"
---

Welcome back to the **Selenium C# Mastery Series**!

In our previous blog, we explored how to handle dropdowns using the `SelectElement` class. Today, we are tackling two more incredibly common form components: **Checkboxes and Radio Buttons**.

While clicking them is simple, the logic behind *when* and *why* to click them requires a bit of strategy.

---

### Understanding the `.Selected` Property

When automating checkboxes and radio buttons, you cannot just indiscriminately call `.Click()`. 

Why? Because if a checkbox is *already checked*, clicking it again will *uncheck* it! 

To prevent this, Selenium provides the **`.Selected`** property on the `IWebElement` object. This property returns a boolean (`true` or `false`) indicating whether the element is currently checked.

**Golden Rule:** Always check the `.Selected` state before clicking a checkbox!

### Handling Checkboxes and Radio Buttons in C#

Let's write a script! We will navigate to the MyCodeYatra Form Practice Sandbox and automate the Gender radio buttons and the Interests checkboxes.

To keep our repository organized, we will create a new class called `Blog5_CheckboxesRadios`. Here is the complete code:

```csharp
using System;
using OpenQA.Selenium;
using OpenQA.Selenium.Chrome;
using System.Threading;
using System.Collections.ObjectModel;
namespace SeleniumCSharpTutorial
{
    public class Blog5_CheckboxesRadios
    {
        public static void Run()
        {
            Console.WriteLine("Initializing ChromeDriver for Checkboxes and Radio Buttons...");
            ChromeOptions options = new ChromeOptions();
            options.AddArgument("--headless");
            options.AddArgument("--disable-gpu");
            options.AddArgument("--window-size=1920,1080");
            using (IWebDriver driver = new ChromeDriver(options))
            {
                driver.Navigate().GoToUrl("https://practice.mycodeyatra.com/#/form-practice");
                Console.WriteLine("Navigated to Form Practice Site.");
                Thread.Sleep(2000); 
                Console.WriteLine("\n--- HANDLING RADIO BUTTONS ---");
                // Locate all Gender radio buttons by Name
                ReadOnlyCollection<IWebElement> genderRadios = driver.FindElements(By.Name("gender"));
                foreach (var radio in genderRadios)
                {
                    if (radio.GetAttribute("value") == "Female")
                    {
                        Console.WriteLine("Selecting 'Female' radio button...");
                        radio.Click();
                        Console.WriteLine("Is 'Female' selected? " + radio.Selected);
                    }
                }
                Console.WriteLine("\n--- HANDLING CHECKBOXES ---");
                // Locate the 'Automation' and 'Testing' checkboxes
                IWebElement automationCheckbox = driver.FindElement(By.XPath("//input[@value='Automation']"));
                IWebElement testingCheckbox = driver.FindElement(By.XPath("//input[@value='Testing']"));
                Console.WriteLine("Is 'Automation' already selected? " + automationCheckbox.Selected);
                // Smart Clicking: Only click if it is NOT selected
                if (!automationCheckbox.Selected)
                {
                    Console.WriteLine("Clicking 'Automation' checkbox...");
                    automationCheckbox.Click();
                }
                if (!testingCheckbox.Selected)
                {
                    Console.WriteLine("Clicking 'Testing' checkbox...");
                    testingCheckbox.Click();
                }
                Console.WriteLine("Is 'Automation' selected now? " + automationCheckbox.Selected);
                Console.WriteLine("Is 'Testing' selected now? " + testingCheckbox.Selected);
                Console.WriteLine("\nUnchecking the 'Automation' checkbox...");
                // Checkboxes can be unchecked by clicking them again!
                if (automationCheckbox.Selected)
                {
                    automationCheckbox.Click();
                }
                Console.WriteLine("Is 'Automation' selected after unchecking? " + automationCheckbox.Selected);
                Console.WriteLine("Checkboxes and Radio Buttons script executed successfully!");
            }
        }
    }
}
```

> **💡 Pro Tip:** Notice how we used `driver.FindElements` (plural) for the radio buttons? Radio buttons share the exact same `name` attribute! By fetching them as a collection, we can iterate through them and click the specific one we want based on its `value`.

---

### Executing the Code

Run your C# application using the terminal:

```bash
dotnet run
```

### 🎯 Console Output

```text
Initializing ChromeDriver for Checkboxes and Radio Buttons...
Navigated to Form Practice Site.

--- HANDLING RADIO BUTTONS ---
Selecting 'Female' radio button...
Is 'Female' selected? True

--- HANDLING CHECKBOXES ---
Is 'Automation' already selected? False
Clicking 'Automation' checkbox...
Clicking 'Testing' checkbox...
Is 'Automation' selected now? True
Is 'Testing' selected now? True

Unchecking the 'Automation' checkbox...
Is 'Automation' selected after unchecking? False
Checkboxes and Radio Buttons script executed successfully!
```

### Summary

Congratulations! You now have a foolproof strategy for interacting with checkboxes and radio buttons. By leveraging `FindElements` and the `.Selected` property, your scripts will be significantly more reliable.

In the next blog, we will cover one of the most critical concepts in all of UI Automation: **Wait Strategies (WebDriverWait)**!

Keep practicing, and see you in the next tutorial!
