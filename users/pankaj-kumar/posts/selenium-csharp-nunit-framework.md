# The Testing Framework: NUnit in Selenium C#

Welcome to Phase 2 of our Selenium C# Mastery series! Until now, we have been running our Selenium scripts inside a standard `Console Application` using a simple `Main` method. While this works for learning basic interactions, real-world test automation requires a dedicated **Testing Framework** to structure, manage, and execute tests effectively.

In the .NET ecosystem, **NUnit** is one of the most popular open-source testing frameworks. It allows us to structure tests logically, run multiple tests seamlessly, and generate reports.

In this tutorial, we will explore:
1. Why we need NUnit.
2. How to install and set up NUnit in our project.
3. The core NUnit Attributes: `[TestFixture]`, `[SetUp]`, `[Test]`, and `[TearDown]`.
4. Writing our first structured Selenium test using NUnit.

---

## 1. Why Do We Need NUnit?

If you try to run dozens of test cases inside a simple `Main` method, your code will quickly become a messy, unmanageable block of `if-else` statements. You will also struggle to track which tests passed and which failed.

A testing framework like NUnit provides several benefits:
- **Test Organization**: Divides tests into logical units.
- **Execution Control**: Decides which tests run, in what order, and allows skipping specific tests.
- **Assertions**: Provides built-in methods to verify expected vs. actual outcomes (we will cover this deeply in the next blog).
- **Test Reporting**: Integrates easily with CI/CD tools to produce test execution reports.

---

## 2. Setting Up NUnit in Our Project

To start using NUnit, we need to install a few NuGet packages. Open your terminal or Package Manager Console and run the following commands:

```bash
dotnet add package NUnit
dotnet add package NUnit3TestAdapter
dotnet add package Microsoft.NET.Test.Sdk
```

- **NUnit**: The core framework.
- **NUnit3TestAdapter**: Allows Visual Studio or `dotnet test` command to discover and run NUnit tests.
- **Microsoft.NET.Test.Sdk**: The underlying testing infrastructure for .NET.

*Note: Since we are in a Console App, we also need to add `<GenerateProgramFile>false</GenerateProgramFile>` in our `.csproj` file to prevent conflicting entry points during test execution.*

---

## 3. Core NUnit Attributes

NUnit uses **Attributes** (similar to annotations in Java) to identify methods that have special meanings. Here are the four most common attributes you will use every day:

1. **`[TestFixture]`**: Placed at the class level. It marks the class as containing tests and setup/teardown methods.
2. **`[SetUp]`**: Placed on a method. This method executes **before each** test method. It is the perfect place to initialize the WebDriver and open the browser.
3. **`[Test]`**: Placed on a method. It marks the method as an actual test case to be executed.
4. **`[TearDown]`**: Placed on a method. This method executes **after each** test method, regardless of whether the test passed or failed. It is used to safely close the browser.

---

## 4. Writing Our First NUnit Test

Let's refactor our standard script into a structured NUnit test class. Create a new file named `Blog11_NUnitFramework.cs`.

```csharp
using NUnit.Framework;
using OpenQA.Selenium;
using OpenQA.Selenium.Chrome;
using System;

namespace mcyt_sel_csharp
{
    // 1. Mark the class as a Test Fixture
    [TestFixture]
    public class Blog11_NUnitFramework
    {
        private IWebDriver driver;

        // 2. Setup method runs BEFORE the test
        [SetUp]
        public void Setup()
        {
            Console.WriteLine("Executing [SetUp]: Initializing ChromeDriver and opening browser.");
            driver = new ChromeDriver();
            driver.Manage().Window.Maximize();
            driver.Manage().Timeouts().ImplicitWait = TimeSpan.FromSeconds(10);
        }

        // 3. The actual Test method
        [Test]
        public void TestFormSubmission()
        {
            Console.WriteLine("Executing [Test]: Navigating to site and filling form.");
            
            // Navigate to practice page
            driver.Navigate().GoToUrl("https://practice.mycodeyatra.com/#/form-practice");

            // Perform Form Fill
            IWebElement nameInput = driver.FindElement(By.Name("fullName"));
            nameInput.SendKeys("Pankaj Kumar");

            IWebElement emailInput = driver.FindElement(By.CssSelector("input[type='email']"));
            emailInput.SendKeys("test@mycodeyatra.com");

            IWebElement submitBtn = driver.FindElement(By.XPath("//button[text()='Submit Form']"));
            submitBtn.Click();

            Console.WriteLine("Test Form Submission Executed Successfully.");
        }

        // 4. TearDown method runs AFTER the test
        [TearDown]
        public void TearDown()
        {
            Console.WriteLine("Executing [TearDown]: Closing the browser.");
            if (driver != null)
            {
                // Always quit the driver to free up resources
                driver.Quit();
                driver.Dispose();
            }
        }
    }
}
```

### How to Run This Test
Instead of running the `Program.cs` file, you can now run tests directly using the Test Explorer in your IDE or by running the following command in your terminal:

```bash
dotnet test
```

When you run `dotnet test`, the output will show that `Blog11_NUnitFramework.TestLoginFunctionality` passed successfully, clearly separating the setup, execution, and teardown phases!

---

## Conclusion

By introducing **NUnit**, we have moved from simple procedural scripts to an organized testing architecture. The `[SetUp]` and `[TearDown]` methods prevent code duplication by handling our browser lifecycle automatically, leaving our `[Test]` method to focus strictly on the test logic.

In the next blog, we will dive into **NUnit Assertions (`Assert.That`)**, which is how we actually verify that our application behaves exactly as expected (e.g., verifying error messages, titles, or element visibility).

Happy Testing!
