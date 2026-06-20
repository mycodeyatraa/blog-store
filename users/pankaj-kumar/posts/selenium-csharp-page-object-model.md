---
title: "Introduction to Page Object Model (POM) in Selenium C#"
date: "13-Jan-2025"
description: "Learn how to structure your Selenium C# test automation framework using the Page Object Model (POM) design pattern for maximum reusability and maintenance."
categories: ["Selenium C#", "Test Automation"]
tags: ["Selenium", "C#", ".NET", "Page Object Model", "POM", "Framework", "Design Pattern"]
author: "Pankaj Kumar"
lastUpdated: "13-Jan-2025"
---

Welcome back to the **Selenium C# Mastery Series**!

Up until now, we have been writing our locators and test actions directly inside our `[Test]` methods. While this works for simple scripts, it becomes a nightmare to maintain as your project grows. If developers change the ID of a button, you would have to manually search and update every single test that interacts with that button.

Enter the **Page Object Model (POM)**.

POM is a widely used design pattern in test automation that encourages the **Separation of Concerns**. It dictates that we should separate our page elements and actions (the "Page Objects") from our actual test logic (the "Test Cases").

---

## 1. Why Use the Page Object Model?

- **Reusability**: You write the locator for an element once, and multiple tests can reuse it.
- **Maintainability**: If the UI changes, you only update the locator in ONE place (the Page class), and all tests instantly inherit the fix.
- **Readability**: Tests become incredibly clean and read almost like plain English.

---

## 2. Setting Up Our First Page Class

Let's create a dedicated class to represent the [MyCodeYatra Practice Form](https://practice.mycodeyatra.com/#/form-practice).

In your project, create a new folder named `Pages`, and inside it, create a file named `PracticeFormPage.cs`.

```cs
using OpenQA.Selenium;
namespace mcyt_sel_csharp.Pages
{
    public class PracticeFormPage
    {
        private IWebDriver driver;
        // Constructor to initialize the driver
        public PracticeFormPage(IWebDriver driver)
        {
            this.driver = driver;
        }
        // 1. Locators
        // We use Private properties/fields to encapsulate our locators
        private By NameInput => By.Name("fullName");
        private By EmailInput => By.CssSelector("input[type='email']");
        private By SubmitButton => By.XPath("//button[text()='Submit Form']");
        // 2. Actions (Methods)
        // We expose public methods that tests can interact with
        public void NavigateTo()
        {
            driver.Navigate().GoToUrl("https://practice.mycodeyatra.com/#/form-practice");
        }
        public void EnterName(string name)
        {
            driver.FindElement(NameInput).SendKeys(name);
        }
        public void EnterEmail(string email)
        {
            driver.FindElement(EmailInput).SendKeys(email);
        }
        public void ClickSubmit()
        {
            driver.FindElement(SubmitButton).Click();
        }
        // Combined business logic method
        public void FillAndSubmitForm(string name, string email)
        {
            EnterName(name);
            EnterEmail(email);
            ClickSubmit();
        }
    }
}
```

Notice how we encapsulated the locators as `private By` fields using Expression-Bodied Members (`=>`). This ensures tests cannot mess with the locators directly.

---

## 3. Writing the NUnit Test with POM

Now, let's create a new test file named `Blog15_PageObjectModel.cs`. 

Notice how clean the test method becomes because all the heavy lifting (locators and clicking) is hidden inside `PracticeFormPage.cs`.

```cs
using NUnit.Framework;
using OpenQA.Selenium;
using OpenQA.Selenium.Chrome;
using System;
using mcyt_sel_csharp.Pages;
namespace mcyt_sel_csharp
{
    [TestFixture]
    public class Blog15_PageObjectModel
    {
        private IWebDriver driver;
        private PracticeFormPage formPage;
        [SetUp]
        public void Setup()
        {
            driver = new ChromeDriver();
            driver.Manage().Window.Maximize();
            driver.Manage().Timeouts().ImplicitWait = TimeSpan.FromSeconds(10);
            // Initialize the Page Object
            formPage = new PracticeFormPage(driver);
        }
        [Test]
        public void TestFormSubmissionUsingPOM()
        {
            Console.WriteLine("Navigating to the Practice Form...");
            formPage.NavigateTo();
            Console.WriteLine("Filling the form using Page Actions...");
            formPage.FillAndSubmitForm("Pankaj Kumar", "pankaj@test.com");
            // Assertions remain in the test class!
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

### The Golden Rule of POM
**Page classes should NEVER contain assertions!**
The Page class is responsible for interactions (clicking, typing) and returning state (getting text). The **Test class** is the only place where `Assert.That()` should live.

---

## 4. Running the Test

Run your suite from the terminal:

```sh
dotnet test
```

**Expected Output:**

```sh
Starting test execution, please wait...
A total of 1 test files matched the specified pattern.
Navigating to the Practice Form...
Filling the form using Page Actions...
Passed!  - Failed:     0, Passed:     1, Skipped:     0, Total:     1, Duration: 5 s - mcyt-sel-csharp.dll (net10.0)
```

---

## Conclusion

By adopting the Page Object Model, you have transformed your framework from a simple scripting tool into a robust, enterprise-grade architecture. From this point forward, every new screen you automate should have its own dedicated Page Class.

In the next blog, we will take POM a step further by introducing the **PageFactory** pattern and the `[FindsBy]` attribute to make element location even more elegant.

Happy Automating!
