---
title: "Generating Beautiful HTML Reports with ExtentReports in Selenium C#"
date: "23-Jan-2025"
description: "Learn how to integrate ExtentReports into your Selenium C# framework to generate beautiful, interactive HTML dashboards for your automation runs."
categories: ["Selenium C#", "Test Automation"]
tags: ["Selenium", "C#", ".NET", "ExtentReports", "Reporting", "NUnit", "Framework Design"]
author: "Pankaj Kumar"
lastUpdated: "23-Jan-2025"
---

Welcome back to the **Selenium C# Mastery Series**!

We have built a powerful, data-driven, and highly robust automation framework. However, there is one crucial piece missing: **Visibility**. 

When your test suite finishes running, NUnit simply outputs a console message saying "Passed" or "Failed." While this is great for developers, Project Managers and QA Leads cannot read terminal outputs. They need visual dashboards, pie charts, and step-by-step logs.

To achieve this, we will integrate the industry standard for test reporting: **ExtentReports**.

---

## 1. Installing ExtentReports

ExtentReports provides beautiful, interactive HTML dashboards out of the box. Open your terminal and install the official NuGet package:

```sh
dotnet add package ExtentReports
```

---

## 2. Integrating ExtentReports into the BaseTest

Since reporting needs to happen globally across all tests, the absolute best place to configure it is within our `BaseTest.cs` class. 

We will use NUnit's `[OneTimeSetUp]` to initialize the HTML report *before* any tests run, and `[OneTimeTearDown]` to flush the report *after* all tests have finished. We'll also update our `[SetUp]` and `[TearDown]` to create individual test entries and log their final status!

Update your `BaseTest.cs` file:

```cs
using NUnit.Framework;
using NUnit.Framework.Interfaces;
using OpenQA.Selenium;
using mcyt_sel_csharp.Core;
using AventStack.ExtentReports;
using AventStack.ExtentReports.Reporter;
using System;
using System.IO;
namespace mcyt_sel_csharp.Tests
{
    public class BaseTest
    {
        protected IWebDriver driver;
        // Static reporting variables so they persist across all test classes
        protected static ExtentReports extent;
        protected ExtentTest test;
        [OneTimeSetUp]
        public void GlobalReportingSetup()
        {
            // 1. Define where to save the report
            string workingDirectory = Environment.CurrentDirectory;
            string projectDirectory = Directory.GetParent(workingDirectory).Parent.Parent.FullName;
            string reportPath = projectDirectory + "//index.html";
            // 2. Attach the Spark Reporter (HTML)
            ExtentSparkReporter sparkReporter = new ExtentSparkReporter(reportPath);
            extent = new ExtentReports();
            extent.AttachReporter(sparkReporter);
            // 3. Add Environment Info
            extent.AddSystemInfo("Environment", ConfigReader.Environment);
            extent.AddSystemInfo("Browser", ConfigReader.Browser);
            extent.AddSystemInfo("Tester", "Pankaj Kumar");
        }
        [SetUp]
        public void Setup()
        {
            // Create a node in the report for the currently executing test
            test = extent.CreateTest(TestContext.CurrentContext.Test.Name);
            BrowserType type = BrowserFactory.GetBrowserType(ConfigReader.Browser);
            driver = BrowserFactory.InitDriver(type);
            driver.Manage().Timeouts().ImplicitWait = TimeSpan.FromSeconds(ConfigReader.ImplicitWait);
            test.Log(Status.Info, $"Browser launched: {ConfigReader.Browser}");
        }
        [TearDown]
        public void TearDown()
        {
            // Log the outcome of the test (Pass/Fail)
            var status = TestContext.CurrentContext.Result.Outcome.Status;
            if (status == TestStatus.Failed)
            {
                test.Fail("Test Failed!");
            }
            else if (status == TestStatus.Passed)
            {
                test.Pass("Test Passed Successfully!");
            }
            if (driver != null)
            {
                driver.Quit();
                driver.Dispose();
            }
        }
        [OneTimeTearDown]
        public void GlobalReportingTearDown()
        {
            // Generate the physical HTML file
            extent.Flush();
        }
    }
}
```

---

## 3. Writing a Test with Logging

Now that our BaseTest handles the reporting engine, we can actively log custom steps inside our actual test methods using the `test` object!

Create `Blog22_ExtentReports.cs`:

```cs
using NUnit.Framework;
using mcyt_sel_csharp.Tests;
using mcyt_sel_csharp.Core;
using AventStack.ExtentReports;
namespace mcyt_sel_csharp
{
    [TestFixture]
    public class Blog22_ExtentReports : BaseTest
    {
        [Test]
        public void TestWithLogging()
        {
            test.Log(Status.Info, $"Navigating to {ConfigReader.BaseUrl}");
            driver.Navigate().GoToUrl(ConfigReader.BaseUrl);
            test.Log(Status.Info, "Verifying URL contains 'practice'");
            Assert.That(driver.Url, Does.Contain("practice"));
        }
    }
}
```

---

## 4. Viewing the Beautiful Results!

Run your tests using the .NET CLI:

```sh
dotnet test --filter "Blog22"
```

Once the test completes, navigate to the root of your C# project in File Explorer. You will see a brand new file named `index.html`. 

Double-click it to open it in your browser! You will see a stunning dashboard featuring:
- A pie chart showing your Pass/Fail ratio.
- The `Environment`, `Browser`, and `Tester` metadata.
- A chronological timeline of every `test.Log()` step you inserted into your code!

---

## Conclusion

With **ExtentReports**, you have completely transformed your framework from a developer-only tool into a robust, stakeholder-friendly QA engine. Anyone on your team can now open `index.html` and instantly understand exactly what your test did and why it passed or failed.

In our next blog, we will take reporting one step further. What happens when a test fails? We need evidence! We will learn how to capture **Screenshots on Failure** and embed them directly into this HTML report.

Happy Automating!
