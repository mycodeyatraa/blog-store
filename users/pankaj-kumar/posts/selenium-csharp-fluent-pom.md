---
title: "Advanced POM: The Fluent Page Object Model (Method Chaining)"
date: "17-Jan-2025"
description: "Take your Selenium C# Page Object Model to the next level by implementing the Fluent Design Pattern for elegant, readable method chaining."
categories: ["Selenium C#", "Test Automation"]
tags: ["Selenium", "C#", ".NET", "Page Object Model", "POM", "Fluent Design", "Method Chaining"]
author: "Pankaj Kumar"
lastUpdated: "17-Jan-2025"
---

Welcome back to the **Selenium C# Mastery Series**!

In our previous blog, we implemented the foundational **Page Object Model (POM)**. It was a massive leap forward in structuring our tests, but we can make it even better. Right now, our test scripts look like this:

```cs
formPage.EnterName("Pankaj");
formPage.EnterEmail("test@test.com");
formPage.ClickSubmit();
```

While functional, this repetitive calling of `formPage` feels a bit clunky. Enter the **Fluent Page Object Model** (often referred to as Method Chaining).

---

## 1. What is the Fluent Design Pattern?

The Fluent Design Pattern is an object-oriented approach where methods return an object instance (usually `this`), allowing you to chain multiple method calls together in a single, flowing statement.

With a Fluent POM, our previous code transforms into this beautiful, readable chain:

```cs
formPage.EnterName("Pankaj")
        .EnterEmail("test@test.com")
        .ClickSubmit();
```

This pattern not only makes tests look like plain English but also enforces valid application workflows (e.g., clicking 'Submit' might return a new `SuccessPage` object instead of `this`).

---

## 2. Refactoring Our Page Class to be Fluent

Let's modify our `PracticeFormPage.cs` class. The key change is that our action methods (which previously had a `void` return type) will now return `PracticeFormPage`.

Open `PracticeFormPage.cs` and update the methods:

```cs
using OpenQA.Selenium;
namespace mcyt_sel_csharp.Pages
{
    public class PracticeFormPage
    {
        private IWebDriver driver;
        public PracticeFormPage(IWebDriver driver)
        {
            this.driver = driver;
        }
        private By NameInput => By.Name("fullName");
        private By EmailInput => By.CssSelector("input[type='email']");
        private By SubmitButton => By.XPath("//button[text()='Submit Form']");
        public PracticeFormPage NavigateTo()
        {
            driver.Navigate().GoToUrl("https://practice.mycodeyatra.com/#/form-practice");
            return this; // Return the current instance
        }
        public PracticeFormPage EnterName(string name)
        {
            driver.FindElement(NameInput).SendKeys(name);
            return this;
        }
        public PracticeFormPage EnterEmail(string email)
        {
            driver.FindElement(EmailInput).SendKeys(email);
            return this;
        }
        // Let's assume clicking submit stays on the same page for this example.
        // If it navigated to a new page, it should return 'new SuccessPage(driver);'
        public PracticeFormPage ClickSubmit()
        {
            driver.FindElement(SubmitButton).Click();
            return this; 
        }
    }
}
```

Notice the crucial addition: `return this;`. This simple line is the secret engine behind method chaining!

---

## 3. Writing the Fluent Test

Now, create a new test file named `Blog16_FluentPOM.cs`. Watch how elegantly our test method reads:

```cs
using NUnit.Framework;
using OpenQA.Selenium;
using OpenQA.Selenium.Chrome;
using System;
using mcyt_sel_csharp.Pages;
namespace mcyt_sel_csharp
{
    [TestFixture]
    public class Blog16_FluentPOM
    {
        private IWebDriver driver;
        [SetUp]
        public void Setup()
        {
            driver = new ChromeDriver();
            driver.Manage().Window.Maximize();
            driver.Manage().Timeouts().ImplicitWait = TimeSpan.FromSeconds(10);
        }
        [Test]
        public void TestFormSubmissionFluently()
        {
            Console.WriteLine("Executing Fluent Page Object Test...");
            PracticeFormPage formPage = new PracticeFormPage(driver);
            // The Fluent Chain
            formPage.NavigateTo()
                    .EnterName("Pankaj Kumar")
                    .EnterEmail("pankaj@fluent.com")
                    .ClickSubmit();
            // The assertion
            Assert.That(driver.Url, Does.Contain("form-practice"), "Form URL mismatch!");
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

## 4. Running the Test

Run the test using the .NET CLI:

```sh
dotnet test --filter "Blog16"
```

**Expected Output:**

```sh
Starting test execution, please wait...
A total of 1 test files matched the specified pattern.
Executing Fluent Page Object Test...
Passed!  - Failed:     0, Passed:     1, Skipped:     0, Total:     1, Duration: 4 s - mcyt-sel-csharp.dll (net10.0)
```

---

## Conclusion

The Fluent Page Object Model bridges the gap between raw code and human-readable test steps. It makes your test cases self-documenting and significantly reduces boilerplate code.

While `PageFactory` used to be the industry standard years ago, it is now officially deprecated. The Fluent POM pattern is the modern, enterprise-approved way to build maintainable Selenium C# automation frameworks!

In our next blog, we will cover **Advanced Wait Strategies** specifically tailored for handling dynamic elements within the Page Object Model.

Happy Automating!
