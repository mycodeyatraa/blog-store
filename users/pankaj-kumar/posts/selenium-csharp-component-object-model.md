---
title: "Component Object Model (COM): Reusable Widgets in Selenium C#"
date: "19-Jan-2025"
description: "Evolve your Selenium C# framework by breaking down Page Objects into reusable UI Components (like Navbars and Footers) using the Component Object Model."
categories: ["Selenium C#", "Test Automation"]
tags: ["Selenium", "C#", ".NET", "Component Object Model", "COM", "POM", "Framework Design"]
author: "Pankaj Kumar"
lastUpdated: "19-Jan-2025"
---

Welcome back to the **Selenium C# Mastery Series**!

We have successfully built a robust Page Object Model (POM) architecture using Fluent Method Chaining. However, as your application grows, you will notice a glaring problem: **Duplication across pages**.

Almost every web application has a global **Navigation Bar**, a **Footer**, or a reusable **Modal/Dialog**. If you duplicate the locators for the Navbar across *every single Page Class* (e.g., `HomePage`, `LoginPage`, `DashboardPage`), you violate the DRY (Don't Repeat Yourself) principle.

To solve this, we introduce the **Component Object Model (COM)**.

---

## 1. What is the Component Object Model?

COM is a natural evolution of POM. Instead of modeling an entire webpage as a single class, we model **reusable sections** of a webpage (Components) as their own classes.

A Page Class is then composed of these Components!

---

## 2. Creating a Reusable Component

Let's build a `NavbarComponent` that represents the top navigation bar of the MyCodeYatra practice site.

Create a new folder named `Components` and add `NavbarComponent.cs`:

```cs
using OpenQA.Selenium;
namespace mcyt_sel_csharp.Components
{
    public class NavbarComponent
    {
        private IWebDriver driver;
        public NavbarComponent(IWebDriver driver)
        {
            this.driver = driver;
        }
        // Locators restricted to the Navbar scope
        private By HomeLink => By.XPath("//a[text()='Home']");
        private By PracticeLink => By.XPath("//a[text()='Practice']");
        private By LoginButton => By.XPath("//button[text()='Login']");
        // Component Actions
        public void ClickHome()
        {
            driver.FindElement(HomeLink).Click();
        }
        public void ClickPractice()
        {
            driver.FindElement(PracticeLink).Click();
        }
        public void ClickLogin()
        {
            driver.FindElement(LoginButton).Click();
        }
    }
}
```

---

## 3. Integrating the Component into a Page

Now, how do we use this? Every Page Class that *contains* the Navbar simply instantiates the `NavbarComponent` as a property!

Let's create a simplified `BasePage.cs` that all our future pages will inherit from. Since the Navbar is on *every* page, it belongs in the Base Page!

```cs
using OpenQA.Selenium;
using mcyt_sel_csharp.Components;
namespace mcyt_sel_csharp.Pages
{
    public class BasePage
    {
        protected IWebDriver driver;
        // Expose the Navbar Component!
        public NavbarComponent Navbar { get; private set; }
        public BasePage(IWebDriver driver)
        {
            this.driver = driver;
            this.Navbar = new NavbarComponent(driver);
        }
    }
}
```

Now, let's create a mock `HomePage` that inherits from `BasePage`:

```cs
using OpenQA.Selenium;
namespace mcyt_sel_csharp.Pages
{
    public class HomePage : BasePage
    {
        public HomePage(IWebDriver driver) : base(driver)
        {
        }
        public HomePage NavigateTo()
        {
            driver.Navigate().GoToUrl("https://practice.mycodeyatra.com/");
            return this;
        }
    }
}
```

---

## 4. Writing the Component-Driven Test

Create a new test file named `Blog18_ComponentObjectModel.cs`. Watch how intuitive the code becomes:

```cs
using NUnit.Framework;
using OpenQA.Selenium;
using OpenQA.Selenium.Chrome;
using mcyt_sel_csharp.Pages;
namespace mcyt_sel_csharp
{
    [TestFixture]
    public class Blog18_ComponentObjectModel
    {
        private IWebDriver driver;
        [SetUp]
        public void Setup()
        {
            driver = new ChromeDriver();
            driver.Manage().Window.Maximize();
        }
        [Test]
        public void TestNavbarComponent()
        {
            // 1. Initialize our Page
            HomePage homePage = new HomePage(driver);
            homePage.NavigateTo();
            // 2. Interact with the Navbar Component seamlessly!
            homePage.Navbar.ClickPractice();
            // 3. Assert navigation was successful
            Assert.That(driver.Url, Does.Contain("practice"));
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

## Conclusion

The **Component Object Model (COM)** allows you to build modular, highly scalable automation frameworks. By creating classes for `NavbarComponent`, `FooterComponent`, or `DatePickerComponent`, you guarantee that locator changes are isolated to a single file, keeping your test maintenance near zero!

In the next blog, we will discuss **Cross-Browser Testing** and how to design a WebDriver Factory to run these beautiful POM/COM tests across Chrome, Firefox, and Edge!

Happy Automating!
