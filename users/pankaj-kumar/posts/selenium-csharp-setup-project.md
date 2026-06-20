---
title: "Selenium C# Mastery: Setup Your First Automation Project"
date: "02-Jan-2025"
description: "Learn how to set up your first Selenium C# (.NET) project from scratch using NuGet, Visual Studio, and ChromeDriver."
categories: ["Selenium C#", "Test Automation"]
tags: ["Selenium", "C#", ".NET", "NuGet", "ChromeDriver"]
author: "Pankaj Kumar"
lastUpdated: "02-Jan-2025"
---

Welcome to the **Selenium C# (.NET) Mastery Series**!

C# is the backbone of modern enterprise applications, especially within the Microsoft ecosystem. In this tutorial, we are going to build our very first Selenium automation project using C# and .NET.

By the end of this blog, you will have successfully instantiated `ChromeDriver`, navigated to the MyCodeYatra practice site, and retrieved the page title dynamically using code!

---

### Step 1: Create a New C# Console Project

First, open your terminal (or Visual Studio) and create a brand new C# Console application. If you are using the .NET CLI, run the following commands:

```bash
mkdir mcyt-sel-csharp
cd mcyt-sel-csharp
dotnet new console
```

This generates a minimal `Program.cs` file and a `.csproj` file which we will use to manage our dependencies.

---

### Step 2: Install Selenium via NuGet

Unlike Java (which uses Maven) or Python (which uses pip), the C# ecosystem relies on **NuGet** for package management. We need two core packages to make Selenium work:

1. **`Selenium.WebDriver`**: The core API bindings for C#.
2. **`Selenium.WebDriver.ChromeDriver`**: The binary required to control the Chrome browser.

Run the following commands to install them:

```bash
dotnet add package Selenium.WebDriver
dotnet add package Selenium.WebDriver.ChromeDriver
```

---

### Step 3: Write Your First Selenium C# Script

Open the `Program.cs` file. We are going to write a script that launches a Chrome browser in headless mode, navigates to the official MyCodeYatra Practice Sandbox, and prints the title of the page to the console.

```csharp
using System;
using OpenQA.Selenium;
using OpenQA.Selenium.Chrome;
namespace SeleniumCSharpTutorial
{
    class Program
    {
        static void Main(string[] args)
        {
            Console.WriteLine("Initializing ChromeDriver...");
            // Step 1: Set up ChromeOptions
            ChromeOptions options = new ChromeOptions();
            options.AddArgument("--headless"); // Run without a UI
            options.AddArgument("--disable-gpu");
            options.AddArgument("--window-size=1920,1080");
            // Step 2: Instantiate the WebDriver using a 'using' statement for memory management
            using (IWebDriver driver = new ChromeDriver(options))
            {
                Console.WriteLine("Navigating to MyCodeYatra Practice Site...");
                // Step 3: Navigate to the target URL
                driver.Navigate().GoToUrl("https://practice.mycodeyatra.com/");
                // Step 4: Fetch the page title
                string pageTitle = driver.Title;
                Console.WriteLine("======================================");
                Console.WriteLine("SUCCESS! Page Title Retrieved: " + pageTitle");
                Console.WriteLine("======================================");
            } // Browser automatically closes and cleans up memory here
            Console.WriteLine("Browser closed successfully.");
        }
    }
}
```

> **💡 Pro Tip (The `using` block):** In C#, `IWebDriver` implements `IDisposable`. By wrapping our driver instantiation in a `using` block, C# will automatically invoke `driver.Dispose()` and clean up the memory/process when the block finishes, preventing memory leaks!

---

### Step 4: Execute the Code and Verify Output

Now, let's run the project and see the magic happen! In your terminal, execute:

```bash
dotnet run
```

### 🎯 Console Output

```text
Initializing ChromeDriver...
Navigating to MyCodeYatra Practice Site...
======================================
SUCCESS! Page Title Retrieved: MyCodeYatra | Test Automation Sandbox
======================================
Browser closed successfully.
```

### Summary

Congratulations! You have successfully established a pristine Selenium C# architecture using NuGet, written your first navigation script, and verified the output.

In the next blog, we will dive deep into **Locators in C#**, where we will learn how to pinpoint exact web elements using XPath, CSS Selectors, IDs, and Names.

Stay tuned, and happy coding!
