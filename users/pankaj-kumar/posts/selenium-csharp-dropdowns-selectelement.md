---
title: "Handling Dropdowns in Selenium C# with SelectElement"
date: "05-Jan-2025"
description: "Master dropdown interactions in Selenium C#. Learn how to use the SelectElement class to select options by text, value, and index, including multi-select dropdowns."
categories: ["Selenium C#", "Test Automation"]
tags: ["Selenium", "C#", ".NET", "Dropdown", "SelectElement", "Multi-Select"]
author: "Pankaj Kumar"
lastUpdated: "05-Jan-2025"
---

Welcome back to the **Selenium C# Mastery Series**!

So far, we have successfully tackled clicking buttons and typing into text boxes. But what happens when you encounter a dropdown menu? 

In HTML, standard dropdown menus are created using the `<select>` tag. While you *could* try clicking the dropdown and then clicking the option, Selenium provides a much cleaner, more robust solution: the **`SelectElement`** class!

---

### Adding the Selenium.Support Package

Before we can use `SelectElement`, we need to install an additional NuGet package. Open your terminal in your project directory (`mcyt-sel-csharp`) and run:

```bash
dotnet add package Selenium.Support
```

This package contains the `OpenQA.Selenium.Support.UI` namespace, which gives us access to advanced UI helpers, including dropdown management!

### The Three Ways to Select

When dealing with a dropdown, the `SelectElement` class gives you three powerful methods:

1. **`SelectByText(string text)`**: Selects the option that exactly matches the visible text on the screen. (Most recommended!)
2. **`SelectByValue(string value)`**: Selects the option based on its underlying HTML `value` attribute.
3. **`SelectByIndex(int index)`**: Selects the option based on its position. Remember, it is zero-indexed, meaning `0` is the first item.

### Handling Dropdowns in C#

Let's write a script! We will navigate to the MyCodeYatra Form Practice Sandbox and interact with both a standard dropdown (Country) and a multi-select dropdown (Preferred Tools).

To keep our repository organized, we will create a new class called `Blog4_Dropdowns`. Here is the complete code:

```csharp
using System;
using OpenQA.Selenium;
using OpenQA.Selenium.Chrome;
using OpenQA.Selenium.Support.UI;
using System.Threading;
using System.Collections.Generic;
namespace SeleniumCSharpTutorial
{
    public class Blog4_Dropdowns
    {
        public static void Run()
        {
            Console.WriteLine("Initializing ChromeDriver for Dropdowns...");
            ChromeOptions options = new ChromeOptions();
            options.AddArgument("--headless");
            options.AddArgument("--disable-gpu");
            options.AddArgument("--window-size=1920,1080");
            using (IWebDriver driver = new ChromeDriver(options))
            {
                driver.Navigate().GoToUrl("https://practice.mycodeyatra.com/#/form-practice");
                Console.WriteLine("Navigated to Form Practice Site.");
                Thread.Sleep(2000); 
                // 1. Locate the single-select Country Dropdown
                IWebElement countryDropdownElement = driver.FindElement(By.Name("country"));
                // 2. Wrap it inside SelectElement
                SelectElement countrySelect = new SelectElement(countryDropdownElement);
                // Select by Text
                Console.WriteLine("Selecting 'India' by Text...");
                countrySelect.SelectByText("India");
                // Read selected option
                Console.WriteLine("Currently Selected Country: " + countrySelect.SelectedOption.Text);
                // Select by Value
                Console.WriteLine("Selecting 'Canada' by Value...");
                countrySelect.SelectByValue("Canada");
                // Select by Index
                Console.WriteLine("Selecting by Index (1)...");
                countrySelect.SelectByIndex(1);
                Console.WriteLine("Currently Selected Country: " + countrySelect.SelectedOption.Text);
                Console.WriteLine("\n--- MULTI-SELECT DROPDOWN ---");
                // 3. Locate the Multi-Select Dropdown
                IWebElement toolsDropdownElement = driver.FindElement(By.Name("preferredTools"));
                SelectElement toolsSelect = new SelectElement(toolsDropdownElement);
                if (toolsSelect.IsMultiple)
                {
                    Console.WriteLine("Selecting 'Selenium' and 'Playwright'...");
                    toolsSelect.SelectByValue("Selenium");
                    toolsSelect.SelectByText("Playwright");
                    // List all selected options
                    IList<IWebElement> selectedTools = toolsSelect.AllSelectedOptions;
                    Console.WriteLine("Total selected tools: " + selectedTools.Count);
                    foreach (var tool in selectedTools)
                    {
                        Console.WriteLine("- " + tool.Text);
                    }
                    // Deselecting
                    Console.WriteLine("Deselecting 'Selenium'...");
                    toolsSelect.DeselectByValue("Selenium");
                    Console.WriteLine("Total selected tools after deselect: " + toolsSelect.AllSelectedOptions.Count);
                }
                Console.WriteLine("Dropdowns script executed successfully!");
            }
        }
    }
}
```

> **💡 Note:** Notice how we first locate the element as a standard `IWebElement`, and then pass it into the `new SelectElement(...)` constructor! Also, for multi-select dropdowns, we can use `DeselectByValue()` to undo a selection.

### Updating Program.cs

Because we created a separate class, make sure your `Program.cs` `Main` method looks like this so it executes our new logic:

```csharp
using System;
namespace SeleniumCSharpTutorial
{
    class Program
    {
        static void Main(string[] args)
        {
            Blog4_Dropdowns.Run();
        }
    }
}
```

---

### Executing the Code

Run your C# application using the terminal:

```bash
dotnet run
```

### 🎯 Console Output

```text
Initializing ChromeDriver for Dropdowns...
Navigated to Form Practice Site.
Selecting 'India' by Text...
Currently Selected Country: India
Selecting 'Canada' by Value...
Selecting by Index (1)...
Currently Selected Country: United States

--- MULTI-SELECT DROPDOWN ---
Selecting 'Selenium' and 'Playwright'...
Total selected tools: 2
- Selenium
- Playwright
Deselecting 'Selenium'...
Total selected tools after deselect: 1
Dropdowns script executed successfully!
```

### Summary

Awesome job! You have learned how to harness the `Selenium.Support` library to elegantly interact with standard HTML `<select>` dropdowns and multi-select lists. 

In the next blog, we will conquer two more unique UI components: **Checkboxes and Radio Buttons**.

Keep up the great work, and happy coding!
