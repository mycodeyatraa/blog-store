# Mastering NUnit Assertions (`Assert.That`) in Selenium C#

Welcome to Blog 12 of our Selenium C# Mastery series! Now that we have NUnit configured in our project, it's time to learn the most crucial aspect of testing: **Assertions**. 

If a test does not have an assertion, it is not a test—it is simply a script executing actions. Assertions are how we verify that an application is behaving exactly as expected. If the assertion passes, the test passes. If the assertion fails, the test stops executing immediately and is marked as failed.

NUnit provides multiple assertion models, but the modern and most expressive one is the **Constraint Model**, which uses `Assert.That(...)`.

---

## Why Use `Assert.That`?

Historically, developers used classic assertions like `Assert.AreEqual()`, `Assert.IsTrue()`, etc. However, NUnit strongly recommends using `Assert.That()` paired with **Is** and **Does** constraints because it reads like plain English.

Compare these two:
- **Classic**: `Assert.AreEqual("MyCodeYatra | Test Automation Sandbox", driver.Title);`
- **Constraint**: `Assert.That(driver.Title, Is.EqualTo("MyCodeYatra | Test Automation Sandbox"));`

The constraint model is more readable, highly extensible, and provides better error messages when a test fails.

---

## Key Assertions You Will Use

Let's look at the most common assertion types you will need when working with Selenium.

### 1. Equality (`Is.EqualTo`)
Used to verify exact matches. Commonly used for verifying Page Titles, Header Texts, or calculated values.

```csharp
Assert.That(driver.Title, Is.EqualTo("MyCodeYatra | Test Automation Sandbox"), "Title mismatch!");
```

*Note: The third argument is an optional custom failure message.*

### 2. Substrings (`Does.Contain`)
Used when you only want to verify that a partial string exists. Commonly used for verifying URLs or error messages.

```csharp
Assert.That(driver.Url, Does.Contain("form-practice"));
```

### 3. Boolean Conditions (`Is.True` / `Is.False`)
Used to check boolean properties of WebElements (e.g., `.Displayed`, `.Enabled`, `.Selected`).

```csharp
Assert.That(submitButton.Displayed, Is.True, "Button should be visible.");
Assert.That(checkbox.Selected, Is.False, "Checkbox should be unchecked by default.");
```

### 4. Null Checks (`Is.Null` / `Is.Not.Null`)
Used to ensure an object exists.

```csharp
Assert.That(myElement, Is.Not.Null);
```

---

## Putting It All Together: A Complete Test Class

Let's write a complete NUnit test class that navigates to the practice site and asserts various conditions.

Create a file named `Blog12_NUnitAssertions.cs`:

```csharp
using NUnit.Framework;
using OpenQA.Selenium;
using OpenQA.Selenium.Chrome;
using System;
namespace mcyt_sel_csharp
{
    [TestFixture]
    public class Blog12_NUnitAssertions
    {
        private IWebDriver driver;
        [SetUp]
        public void Setup()
        {
            Console.WriteLine("Executing [SetUp]: Initializing ChromeDriver...");
            driver = new ChromeDriver();
            driver.Manage().Window.Maximize();
            driver.Manage().Timeouts().ImplicitWait = TimeSpan.FromSeconds(10);
            // Navigate to our practice form for assertions
            driver.Navigate().GoToUrl("https://practice.mycodeyatra.com/#/form-practice");
        }
        [Test]
        public void TestPageTitleAndUrl()
        {
            Console.WriteLine("Executing Test: Verifying Page Title and URL");
            string currentTitle = driver.Title;
            string currentUrl = driver.Url;
            // 1. Asserting Equality
            Assert.That(currentTitle, Is.EqualTo("MyCodeYatra | Test Automation Sandbox"), "Page title did not match!");
            // 2. Asserting Substring (Contains)
            Assert.That(currentUrl, Does.Contain("form-practice"), "URL does not contain expected path.");
        }
        [Test]
        public void TestElementVisibilityAndState()
        {
            Console.WriteLine("Executing Test: Verifying Element State");
            IWebElement submitBtn = driver.FindElement(By.XPath("//button[text()='Submit Form']"));
            IWebElement nameInput = driver.FindElement(By.Name("fullName"));
            // 3. Asserting True/False conditions
            Assert.That(submitBtn.Displayed, Is.True, "Submit button should be visible on the page.");
            Assert.That(nameInput.Enabled, Is.True, "Name input field should be enabled.");
        }
        [Test]
        public void TestTextContent()
        {
            Console.WriteLine("Executing Test: Verifying Text Content");
            IWebElement header = driver.FindElement(By.TagName("h2"));
            // 4. Asserting Text (Case Insensitive)
            // You can chain constraints like .IgnoreCase for robust assertions!
            Assert.That(header.Text, Is.EqualTo("Form Submission & Data Validation").IgnoreCase, "Header text did not match expected value.");
        }
        [TearDown]
        public void TearDown()
        {
            Console.WriteLine("Executing [TearDown]: Closing browser...");
            if (driver != null)
            {
                driver.Quit();
                driver.Dispose();
            }
        }
    }
}
```

### Running the Tests
Run the tests via the terminal using:

```bash
dotnet test
```

**Expected Output:**

```text
Starting test execution, please wait...
A total of 1 test files matched the specified pattern.
Passed!  - Failed:     0, Passed:     3, Skipped:     0, Total:     3, Duration: 6 s - mcyt-sel-csharp.dll (net10.0)
```

If any of these assertions fail (for instance, if we expected the title to be "Wrong Title"), the test execution would halt at that exact line, and the terminal would clearly state the expected versus actual values alongside our custom failure message.

---

## Conclusion

Assertions are the core of automation testing. With NUnit's `Assert.That` constraint model, you can write highly readable and robust verifications for your Selenium scripts. 

In the next blog, we will transition into **Data-Driven Testing (DDT)**, showing you how to run the same NUnit test multiple times automatically using different sets of data!

Happy Testing!
