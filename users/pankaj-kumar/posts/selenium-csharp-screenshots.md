---
title: "Capturing Screenshots on Failure and Embedding in ExtentReports: Selenium C#"
date: "24-Jan-2025"
description: "Learn how to automatically capture screenshots when a Selenium C# test fails and embed them directly into your ExtentReports HTML dashboard for immediate visual debugging."
categories: ["Selenium C#", "Test Automation"]
tags: ["Selenium", "C#", ".NET", "ExtentReports", "Screenshots", "Debugging", "Framework Design"]
author: "Pankaj Kumar"
lastUpdated: "24-Jan-2025"
---

Welcome back to the **Selenium C# Mastery Series**!

In our last blog, we integrated **ExtentReports** to generate beautiful HTML dashboards. However, when a test fails, reading a stack trace in an HTML report is often not enough. To truly understand *why* a test failed (e.g., did a popup block the button? Did the page fail to load?), we need **visual evidence**.

Today, we are going to configure our framework to **automatically take a screenshot the exact millisecond a test fails**, and embed that image directly into our HTML Report!

---

## 1. Creating a Screenshot Utility

Let's start by creating a helper method that actually takes the screenshot and saves it to a folder. We will create a `Utils` folder in our project and add a `ScreenshotUtility.cs` class.

Selenium provides an interface called `ITakesScreenshot`. By casting our `IWebDriver` to this interface, we can capture the screen.

```cs
using OpenQA.Selenium;
using System;
using System.IO;
namespace mcyt_sel_csharp.Utils
{
    public static class ScreenshotUtility
    {
        public static string TakeScreenshot(IWebDriver driver, string testName)
        {
            // 1. Cast the driver to ITakesScreenshot
            ITakesScreenshot ts = (ITakesScreenshot)driver;
            // 2. Capture the screenshot
            Screenshot screenshot = ts.GetScreenshot();
            // 3. Define the path to save it
            string workingDirectory = Environment.CurrentDirectory;
            string projectDirectory = Directory.GetParent(workingDirectory).Parent.Parent.FullName;
            string screenshotFolder = Path.Combine(projectDirectory, "Screenshots");
            // Create the folder if it doesn't exist
            if (!Directory.Exists(screenshotFolder))
            {
                Directory.CreateDirectory(screenshotFolder);
            }
            // 4. Create a unique filename using timestamp
            string timeStamp = DateTime.Now.ToString("yyyyMMdd_HHmmss");
            string fileName = $"{testName}_{timeStamp}.png";
            string filePath = Path.Combine(screenshotFolder, fileName);
            // 5. Save the file
            screenshot.SaveAsFile(filePath);
            // 6. Return the path so ExtentReports can attach it!
            return filePath;
        }
    }
}
```

---

## 2. Integrating with the BaseTest

Now, we need to wire this utility into our `BaseTest.cs`. 

Remember, our `[TearDown]` method in `BaseTest` executes *after* every single test. This is the absolute perfect place to check if the test failed, and if so, trigger the screenshot!

Update the `[TearDown]` method in your `BaseTest.cs`:

```cs
using mcyt_sel_csharp.Utils; // Don't forget the import!
// ... inside BaseTest class
[TearDown]
public void GlobalTearDown()
{
    var status = TestContext.CurrentContext.Result.Outcome.Status;
    if (status == TestStatus.Failed)
    {
        // 1. Get the error message
        var errorMessage = TestContext.CurrentContext.Result.Message;
        // 2. Take the screenshot!
        string screenshotPath = ScreenshotUtility.TakeScreenshot(driver, TestContext.CurrentContext.Test.Name);
        // 3. Log the failure and attach the image to ExtentReports!
        test.Fail($"Test Failed: {errorMessage}");
        test.AddScreenCaptureFromPath(screenshotPath, "Failure Screenshot");
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
```

---

## 3. Writing a Deliberately Failing Test

To prove this works, let's write a test that we *know* will fail. We will instruct Selenium to look for an element ID that definitely does not exist on the page.

Create `Blog23_ScreenshotOnFailure.cs`:

```cs
using NUnit.Framework;
using OpenQA.Selenium;
using mcyt_sel_csharp.Tests;
using mcyt_sel_csharp.Core;
namespace mcyt_sel_csharp
{
    [TestFixture]
    public class Blog23_ScreenshotOnFailure : BaseTest
    {
        [Test]
        public void TestDeliberateFailure()
        {
            test.Log(AventStack.ExtentReports.Status.Info, "Navigating to practice page.");
            driver.Navigate().GoToUrl(ConfigReader.BaseUrl);
            test.Log(AventStack.ExtentReports.Status.Info, "Attempting to find an element that does not exist!");
            // This will throw a NoSuchElementException and fail the test
            driver.FindElement(By.Id("ThisElementDoesNotExist12345")).Click();
        }
    }
}
```

---

## 4. The Result!

Run the test using your test explorer or the CLI:

```sh
dotnet test --filter "Blog23"
```

1. The test will launch the browser and navigate to the page.
2. It will fail to find the element.
3. NUnit will trigger the `[TearDown]` method.
4. `BaseTest` will detect the `TestStatus.Failed`.
5. It will instantly take a `.png` screenshot and save it to the `/Screenshots` folder.
6. It will embed that image directly into your `index.html` report.

Open your `index.html` file! You will see the red "Failed" badge, the exact Exception Stack Trace, and a clickable thumbnail of the screen exactly as it looked when the test crashed!

---

## Conclusion

By combining the **BaseTest**, **ExtentReports**, and a **Screenshot Utility**, you have created a completely autonomous debugging system. When your tests run overnight in a CI/CD pipeline, you will wake up to an HTML report that shows you exactly what broke, complete with photographic evidence.

In the next blog, we will tackle the final piece of the architectural puzzle: **Reading Test Data from Excel** using EPPlus!

Happy Automating!
