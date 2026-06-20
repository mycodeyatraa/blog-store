---
title: "Data-Driven Testing in Selenium C# with JSON"
date: "12-Jan-2025"
description: "Learn how to implement Data-Driven Testing (DDT) in Selenium C# by reading JSON files using the native System.Text.Json library."
categories: ["Selenium C#", "Test Automation"]
tags: ["Selenium", "C#", ".NET", "Data-Driven Testing", "DDT", "JSON", "System.Text.Json", "NUnit"]
author: "Pankaj Kumar"
lastUpdated: "12-Jan-2025"
---

Welcome back to the **Selenium C# Mastery Series**! 

In our previous blog, we explored Data-Driven Testing using Excel files with EPPlus. While Excel is great for business users, developers and automation engineers often prefer **JSON** (JavaScript Object Notation) for test data. 

JSON is lightweight, native to modern web development, and perfectly integrates with C# without needing any third-party libraries (since .NET Core 3.0+ includes `System.Text.Json` by default).

Let's dive into creating a JSON data-driven test!

---

## 1. Preparing Our JSON Test Data

First, we need to create our JSON data file. Create a file named `TestData.json` at the root of your project folder (right next to your `.csproj` file).

Add the following JSON array containing objects with `FullName` and `Email` properties:

```js
[
  {
    "FullName": "Alex Turner",
    "Email": "alex@test.com"
  },
  {
    "FullName": "Sarah Connor",
    "Email": "sarah@test.com"
  },
  {
    "FullName": "Bruce Wayne",
    "Email": "bruce@test.com"
  }
]
```

---

## 2. Creating the C# Model Class

To easily read this JSON data, we should map it to a C# class. We will use the `[JsonPropertyName]` attribute to ensure the JSON keys match our C# properties exactly.

```cs
using System.Text.Json.Serialization;
namespace mcyt_sel_csharp
{
    // A class to map our JSON objects to C# objects
    public class UserData
    {
        [JsonPropertyName("FullName")]
        public string FullName { get; set; }
        [JsonPropertyName("Email")]
        public string Email { get; set; }
    }
}
```

---

## 3. Writing the Data-Driven Test

Now, let's create our test class named `Blog14_DataDrivenTestingJson.cs`. We will read the `TestData.json` file, deserialize it into a list of our `UserData` objects, and feed it into NUnit's `[TestCaseSource]`.

```cs
using NUnit.Framework;
using OpenQA.Selenium;
using OpenQA.Selenium.Chrome;
using System;
using System.IO;
using System.Collections.Generic;
using System.Text.Json;
using System.Text.Json.Serialization;
namespace mcyt_sel_csharp
{
    public class UserData
    {
        [JsonPropertyName("FullName")]
        public string FullName { get; set; }
        [JsonPropertyName("Email")]
        public string Email { get; set; }
    }
    [TestFixture]
    public class Blog14_DataDrivenTestingJson
    {
        private IWebDriver driver;
        // 1. Method to Read JSON Data dynamically
        public static IEnumerable<TestCaseData> GetJsonData()
        {
            string filePath = Path.Combine(AppDomain.CurrentDomain.BaseDirectory, "../../../TestData.json");
            string jsonContent = File.ReadAllText(filePath);
            // Deserialize the JSON array into a List of UserData objects
            List<UserData> users = JsonSerializer.Deserialize<List<UserData>>(jsonContent);
            foreach (var user in users)
            {
                // Yield returns a new TestCaseData object for each user
                yield return new TestCaseData(user.FullName, user.Email);
            }
        }
        [SetUp]
        public void Setup()
        {
            driver = new ChromeDriver();
            driver.Manage().Window.Maximize();
            driver.Manage().Timeouts().ImplicitWait = TimeSpan.FromSeconds(10);
            driver.Navigate().GoToUrl("https://practice.mycodeyatra.com/#/form-practice");
        }
        // 2. Use TestCaseSource to map our method output directly into the test parameters!
        [Test, TestCaseSource(nameof(GetJsonData))]
        public void TestFormWithJsonData(string fullName, string email)
        {
            Console.WriteLine($"Running Test with JSON Data -> Name: {fullName}, Email: {email}");
            // Locate elements
            IWebElement nameInput = driver.FindElement(By.Name("fullName"));
            IWebElement emailInput = driver.FindElement(By.CssSelector("input[type='email']"));
            IWebElement submitBtn = driver.FindElement(By.XPath("//button[text()='Submit Form']"));
            // Perform actions using dynamic data
            nameInput.SendKeys(fullName);
            emailInput.SendKeys(email);
            submitBtn.Click();
            // Simple assertion
            Assert.That(driver.Url, Does.Contain("form-practice"));
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

### Why JSON Over Excel?
1. **No External Dependencies**: We used `System.Text.Json`, which is built directly into .NET!
2. **Version Control Friendly**: JSON is just text, making it extremely easy to track changes in Git.
3. **Speed**: Reading a text file and parsing JSON is significantly faster than launching an Excel parsing engine.

---

## 4. Running the Test

Run your test suite from the terminal:

```sh
dotnet test
```

**Expected Output:**

```sh
Starting test execution, please wait...
A total of 1 test files matched the specified pattern.
Running Test with JSON Data -> Name: Alex Turner, Email: alex@test.com
Running Test with JSON Data -> Name: Sarah Connor, Email: sarah@test.com
Running Test with JSON Data -> Name: Bruce Wayne, Email: bruce@test.com
Passed!  - Failed:     0, Passed:     3, Skipped:     0, Total:     3, Duration: 10 s - mcyt-sel-csharp.dll (net10.0)
```

You have now successfully run a single test three separate times using JSON data! 

---

## Conclusion

Understanding Data-Driven Testing with both Excel and JSON makes you incredibly versatile as an automation engineer. While Excel is perfect for non-technical stakeholders to provide data, JSON is typically the preferred format for modern developer-centric automation frameworks.

In our next blog, we will introduce the **Page Object Model (POM)** design pattern to completely revolutionize the way we structure our Selenium frameworks.

Happy Automating!
