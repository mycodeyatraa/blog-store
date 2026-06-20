---
title: "Data-Driven Testing in Selenium C# with EPPlus (Excel)"
date: "11-Jan-2025"
description: "Learn how to implement Data-Driven Testing (DDT) in Selenium C# using NUnit's TestCaseSource and EPPlus to read test data dynamically from an Excel file."
categories: ["Selenium C#", "Test Automation"]
tags: ["Selenium", "C#", ".NET", "Data-Driven Testing", "DDT", "EPPlus", "Excel", "NUnit"]
author: "Pankaj Kumar"
lastUpdated: "11-Jan-2025"
---

Welcome back to the **Selenium C# Mastery Series**! 

Up to this point, our tests have been hardcoded. If we wanted to test a login form with 5 different users, we would have had to duplicate our code 5 times. This violates the DRY (Don't Repeat Yourself) principle and makes maintenance a nightmare.

Enter **Data-Driven Testing (DDT)**. DDT allows us to write a single test script and execute it multiple times automatically using different sets of data (e.g., from an Excel file, CSV, or database).

Today, we will learn how to read data from an Excel file using the popular **EPPlus** library and feed it into our NUnit tests using `[TestCaseSource]`.

---

## 1. Setting Up EPPlus

To read Excel files (`.xlsx`), we need an external library. **EPPlus** is the industry standard for handling Excel files in .NET.

Open your terminal and install EPPlus via NuGet (we recommend version 7.x for compatibility with standard tutorials):

```bash
dotnet add package EPPlus --version 7.0.0
```

*Note: Starting from EPPlus 5, the library became commercial. For personal, educational, or internal use, you can set the `LicenseContext` to `NonCommercial`, which we will do in our code!*

---

## 2. Preparing Our Test Data

First, we need an Excel file to read from.

Create an Excel file named `TestData.xlsx` at the root of your project folder (right next to your `.csproj` file). 
Inside the first sheet, add the following data:

| FullName     | Email           |
|--------------|-----------------|
| Pankaj Kumar | pankaj@test.com |
| John Doe     | john@test.com   |
| Jane Smith   | jane@test.com   |

Make sure to save and close the file.

---

## 3. Creating the Data-Driven Test

Now, let's create a new class named `Blog13_DataDrivenTesting.cs`. We will write a utility method to read the Excel file and an NUnit test to consume that data.

```csharp
using NUnit.Framework;
using OpenQA.Selenium;
using OpenQA.Selenium.Chrome;
using OfficeOpenXml;
using System;
using System.IO;
using System.Collections.Generic;
namespace mcyt_sel_csharp
{
    [TestFixture]
    public class Blog13_DataDrivenTesting
    {
        private IWebDriver driver;
        // 1. Method to Read Excel Data dynamically
        public static IEnumerable<TestCaseData> GetExcelData()
        {
            // Set the license context for EPPlus
            ExcelPackage.LicenseContext = LicenseContext.NonCommercial;
            // Define the path to TestData.xlsx
            string filePath = Path.Combine(AppDomain.CurrentDomain.BaseDirectory, "../../../TestData.xlsx");
            using (ExcelPackage package = new ExcelPackage(new FileInfo(filePath)))
            {
                // Access the first worksheet
                ExcelWorksheet worksheet = package.Workbook.Worksheets[0];
                int rowCount = worksheet.Dimension.Rows;
                // Loop through rows (Start from row 2 to skip headers)
                for (int row = 2; row <= rowCount; row++)
                {
                    string fullName = worksheet.Cells[row, 1].Text;
                    string email = worksheet.Cells[row, 2].Text;
                    // Yield returns a new TestCaseData object for each row
                    yield return new TestCaseData(fullName, email);
                }
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
        [Test, TestCaseSource(nameof(GetExcelData))]
        public void TestFormWithExcelData(string fullName, string email)
        {
            Console.WriteLine($"Running Test with Data -> Name: {fullName}, Email: {email}");
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

### Breaking Down the Magic:
1. **`IEnumerable<TestCaseData>`**: This is an NUnit interface. Our method reads the Excel file row by row and `yield returns` a `TestCaseData` object containing the columns.
2. **`[TestCaseSource(nameof(GetExcelData))]`**: This attribute tells NUnit, *"Hey, don't just run this test once. Go execute `GetExcelData()`, and for every `TestCaseData` it yields, run this test again passing those values into the method parameters!"*

---

## 4. Running the Test

Run your test suite from the terminal:

```bash
dotnet test
```

**Expected Output:**

```bash
Starting test execution, please wait...
A total of 1 test files matched the specified pattern.
Running Test with Data -> Name: Pankaj Kumar, Email: pankaj@test.com
Running Test with Data -> Name: John Doe, Email: john@test.com
Running Test with Data -> Name: Jane Smith, Email: jane@test.com
Passed!  - Failed:     0, Passed:     3, Skipped:     0, Total:     3, Duration: 12 s - mcyt-sel-csharp.dll (net10.0)
```

As you can see, even though we only wrote **one** test method (`TestFormWithExcelData`), it executed **three times**-once for each row of data in our Excel sheet!

---

## Conclusion

Data-Driven Testing is an absolute game changer. By separating your test logic from your test data, you make your framework scalable. Need to test 100 different scenarios? Just add 100 rows to the Excel sheet-no code changes required!

In our next blog, we will introduce the **Page Object Model (POM)** design pattern to completely revolutionize the way we structure our Selenium frameworks.

Happy Automating!
