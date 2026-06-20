---
title: "Data-Driven Testing: Reading Test Data from Excel using EPPlus in Selenium C#"
date: "25-Jan-2025"
description: "Learn how to perform Data-Driven Testing by reading credentials from an Excel file (.xlsx) using the powerful EPPlus NuGet package in C#."
categories: ["Selenium C#", "Test Automation"]
tags: ["Selenium", "C#", ".NET", "Data-Driven Testing", "Excel", "EPPlus", "Framework Design"]
author: "Pankaj Kumar"
lastUpdated: "25-Jan-2025"
---

Welcome back to the **Selenium C# Mastery Series**!

In an earlier blog, we learned how to perform Data-Driven Testing by passing data directly into the `[TestCase]` attribute, and then by reading from a JSON file.

However, in the real world, Business Analysts and Product Managers often maintain test data (like thousands of usernames and passwords) in **Excel Spreadsheets (.xlsx)**. Today, we are going to learn how to connect our Selenium framework directly to an Excel file using the most popular C# library for this task: **EPPlus**.

---

## 1. Installing EPPlus

Open your terminal in the root of your project and install the EPPlus NuGet package:

```sh
dotnet add package EPPlus
```

*Note: EPPlus requires you to accept its commercial license terms if used in a commercial product, but it is free for non-commercial/learning use.*

---

## 2. Creating the Excel File

Create a folder named `TestData` in your project root. Inside it, create an Excel file named `LoginData.xlsx` (you can do this via MS Excel, or programmatically if you prefer).

Add some dummy data to the first sheet:

| Username | Password |
| :--- | :--- |
| testuser1@mycodeyatra.com | Password123! |
| invaliduser@mycodeyatra.com | WrongPass! |
| admin@mycodeyatra.com | AdminPass123! |

Make sure to configure your `.csproj` file to copy this Excel file to the output directory during build:

```xml
<ItemGroup>
  <None Update="TestData\LoginData.xlsx">
    <CopyToOutputDirectory>PreserveNewest</CopyToOutputDirectory>
  </None>
</ItemGroup>
```

---

## 3. Building the ExcelReader Utility

Now, let's create a robust utility class inside our `Utils` folder named `ExcelReader.cs`. This class will open the Excel file, read all the rows, and yield them back to NUnit.

```cs
using OfficeOpenXml;
using System;
using System.Collections.Generic;
using System.IO;
namespace mcyt_sel_csharp.Utils
{
    public static class ExcelReader
    {
        public static IEnumerable<object[]> GetLoginData()
        {
            // EPPlus requires you to set the LicenseContext
            ExcelPackage.LicenseContext = LicenseContext.NonCommercial;
            string workingDirectory = Environment.CurrentDirectory;
            string projectDirectory = Directory.GetParent(workingDirectory).Parent.Parent.FullName;
            string filePath = Path.Combine(projectDirectory, "TestData", "LoginData.xlsx");
            FileInfo fileInfo = new FileInfo(filePath);
            using (ExcelPackage package = new ExcelPackage(fileInfo))
            {
                // Get the first worksheet
                ExcelWorksheet worksheet = package.Workbook.Worksheets[0];
                // Get row and column counts
                int rowCount = worksheet.Dimension.Rows;
                int colCount = worksheet.Dimension.Columns;
                // Loop through rows (start at 2 to skip headers!)
                for (int row = 2; row <= rowCount; row++)
                {
                    string username = worksheet.Cells[row, 1].Value?.ToString();
                    string password = worksheet.Cells[row, 2].Value?.ToString();
                    // Yield the data array back to the NUnit test
                    yield return new object[] { username, password };
                }
            }
        }
    }
}
```

### Why use `yield return`?
Using `yield return` allows our method to return one row of data at a time to NUnit without having to build a massive list in memory first. It is highly memory efficient!

---

## 4. Writing the Data-Driven Test

Now that our utility can parse the Excel file, let's write a test that consumes this data using NUnit's `[TestCaseSource]` attribute!

Create `Blog24_ExcelDataDrivenTest.cs`:

```cs
using NUnit.Framework;
using mcyt_sel_csharp.Tests;
using mcyt_sel_csharp.Utils;
using AventStack.ExtentReports;
using System;
namespace mcyt_sel_csharp
{
    [TestFixture]
    public class Blog24_ExcelDataDrivenTest : BaseTest
    {
        [Test]
        // Tell NUnit to pull data from our ExcelReader utility!
        [TestCaseSource(typeof(ExcelReader), nameof(ExcelReader.GetLoginData))]
        public void TestLoginWithExcelData(string username, string password)
        {
            test.Log(Status.Info, $"Attempting login with Username: {username} and Password: {password}");
            // In a real framework, you would use Page Objects here!
            Console.WriteLine($"Testing User: {username}");
            // We just assert true to prove the data was passed successfully
            Assert.That(username, Does.Contain("@mycodeyatra.com"));
        }
    }
}
```

---

## 5. The Result

When you execute `dotnet test`, NUnit will:
1. Call `ExcelReader.GetLoginData()`.
2. Open `LoginData.xlsx` and read the three rows of data.
3. Automatically execute `TestLoginWithExcelData` **three separate times**, injecting a new Username and Password into the test parameters for each run!

Your Extent HTML report will also clearly show three distinct test executions, perfectly logging exactly which email and password were used!

---

## Conclusion

You have successfully integrated Excel parsing into your C# framework! This allows your non-technical team members to write thousands of test permutations in Excel, and your code will execute every single one of them automatically.

In our next blog, we will wrap up our framework design series by discussing **Continuous Integration (CI)**: How to run these tests automatically in **GitHub Actions**!

Happy Automating!
