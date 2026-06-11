---
title: Automating Data-Driven Testing in Selenium: TestNG DataProviders & Apache POI
date: 22-Jul-2024
lastUpdated: 11-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [selenium, java, data-driven-testing, testng-dataprovider, apache-poi, excel-reader]
category: UI Automation
categories: [UI Automation, Selenium, Java]
excerpt: >-
  Master data-driven testing in Selenium WebDriver. Learn how to parse Excel files using Apache POI and feed inputs dynamically to TestNG DataProviders in Java.
readTime: 7 min read
---

# Automating Data-Driven Testing in Selenium: TestNG DataProviders & Apache POI

> 📅 **Last Updated:** 11-Jun-2026

Writing separate test scripts for different sets of input data is highly inefficient. Hardcoding values in your test files limits coverage, bloats code, and makes maintenance a nightmare.

**Data-Driven Testing (DDT)** solves this by separating test data from test logic. Your scripts act as reusable engines that receive parameters from external sources like Excel spreadsheets, databases, or CSV files. In this guide, we will design a robust, clean Data-Driven framework in Java using **TestNG `@DataProvider`** and **Apache POI**.

---

## 🧭 Data-Driven Testing Architecture

The diagram below displays how test parameters are read dynamically from an external spreadsheet, mapped to the TestNG provider stream, and executed in separate browser test loops:

![diagram_1](https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/posts/automating-data-driven-testing-selenium-testng-dataproviders-poi/images/diagram_1.png)

---

> 📥 **Get the Test Data:**
> The complete sample Excel file (`test_data.xlsx`) used in this guide is uploaded to the Git repository. You can view its layout and download it directly here: [test_data.xlsx on GitHub](https://github.com/MYCodeYatra/mcyt-sel-java/blob/main/src/test/resources/test_data.xlsx).

---

## 🛠️ Step-by-Step Code Walkthrough

To parse binary Excel files (`.xlsx` formats), we include the standard Apache POI dependencies in our project's `pom.xml`:

```xml
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi</artifactId>
    <version>5.2.5</version>
</dependency>
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi-ooxml</artifactId>
    <version>5.2.5</version>
</dependency>
```

### 1. Excel Parsing Utility (`ExcelUtil.java`)
First, we build a reusable helper class to open Excel files, navigate to a specific worksheet, and parse row-column data into a 2D Object array. We use POI's `DataFormatter` to automatically translate various cell formats (like numbers or dates) into standard Strings.

```java
package com.mycodeyatra.utils;
import org.apache.poi.ss.usermodel.DataFormatter;
import org.apache.poi.xssf.usermodel.XSSFCell;
import org.apache.poi.xssf.usermodel.XSSFRow;
import org.apache.poi.xssf.usermodel.XSSFSheet;
import org.apache.poi.xssf.usermodel.XSSFWorkbook;
import java.io.FileInputStream;
import java.io.IOException;
public class ExcelUtil {
    public static Object[][] getSheetData(String filePath, String sheetName) {
        Object[][] data = null;
        try (FileInputStream fis = new FileInputStream(filePath);
             XSSFWorkbook workbook = new XSSFWorkbook(fis)) {
            XSSFSheet sheet = workbook.getSheet(sheetName);
            if (sheet == null) {
                throw new IllegalArgumentException("Sheet " + sheetName + " does not exist in the workbook.");
            }
            int rowCount = sheet.getPhysicalNumberOfRows();
            if (rowCount <= 1) {
                return new Object[0][0];
            }
            XSSFRow headerRow = sheet.getRow(0);
            int colCount = headerRow.getLastCellNum();
            // Allocate memory for data (excluding header row)
            data = new Object[rowCount - 1][colCount];
            DataFormatter formatter = new DataFormatter();
            for (int r = 1; r < rowCount; r++) {
                XSSFRow row = sheet.getRow(r);
                if (row == null) continue;
                for (int c = 0; c < colCount; c++) {
                    XSSFCell cell = row.getCell(c);
                    data[r - 1][c] = formatter.formatCellValue(cell);
                }
            }
        } catch (IOException e) {
            System.err.println("Error reading Excel file: " + e.getMessage());
        }
        return data;
    }
}
```

### 2. E2E Test Suite (`DataDrivenTest.java`)
Next, we link our test suite to the utility. We write a `@DataProvider` method that gets the absolute path of `test_data.xlsx` dynamically and invokes `ExcelUtil.getSheetData()`. The `@Test` method is annotated with `dataProvider = "excelUserData"`, instructing TestNG to loop execution for every data row.

```java
package com.mycodeyatra.tests;
import io.github.bonigarcia.wdm.WebDriverManager;
import com.mycodeyatra.utils.ExcelUtil;
import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.chrome.ChromeOptions;
import org.openqa.selenium.support.ui.ExpectedConditions;
import org.openqa.selenium.support.ui.Select;
import org.openqa.selenium.support.ui.WebDriverWait;
import org.testng.Assert;
import org.testng.annotations.AfterMethod;
import org.testng.annotations.BeforeMethod;
import org.testng.annotations.DataProvider;
import org.testng.annotations.Test;
import java.io.File;
import java.time.Duration;
public class DataDrivenTest {
    private WebDriver driver;
    private WebDriverWait wait;
    @BeforeMethod
    public void setUp() {
        WebDriverManager.chromedriver().setup();
        ChromeOptions options = new ChromeOptions();
        options.addArguments("--remote-allow-origins=*");
        options.addArguments("--headless=new");
        options.addArguments("--window-size=1920,1080");
        driver = new ChromeDriver(options);
        driver.manage().window().maximize();
        driver.manage().timeouts().implicitlyWait(Duration.ofSeconds(10));
        wait = new WebDriverWait(driver, Duration.ofSeconds(5));
    }
    @DataProvider(name = "excelUserData")
    public Object[][] getExcelData() {
        String projectPath = System.getProperty("user.dir");
        String excelPath = projectPath + File.separator + "src" + File.separator + "test" 
                + File.separator + "resources" + File.separator + "test_data.xlsx";
        System.out.println("Loading data-driven inputs from Excel path: " + excelPath);
        return ExcelUtil.getSheetData(excelPath, "UserData");
    }
    @Test(dataProvider = "excelUserData")
    public void testFormSubmissionWithMultipleUsers(String name, String email, String phone, 
                                                    String gender, String country, String tool, String bio) {
        String targetUrl = "https://practice.mycodeyatra.com/#/form-practice";
        System.out.println("Executing test iteration for user: " + name + " (" + email + ") on target URL: " + targetUrl);
        driver.get(targetUrl);
        // 1. Fill Text Inputs
        wait.until(ExpectedConditions.presenceOfElementLocated(By.xpath("//input[@data-testid='full-name']"))).sendKeys(name);
        driver.findElement(By.xpath("//input[@data-testid='email']")).sendKeys(email);
        driver.findElement(By.xpath("//input[@data-testid='phone']")).sendKeys(phone);
        // 2. Select Radio Button for Gender dynamically
        String genderTestId = "gender-" + gender.toLowerCase();
        System.out.println("Selecting gender: " + gender + " using locator data-testid=" + genderTestId);
        driver.findElement(By.xpath("//input[@data-testid='" + genderTestId + "']")).click();
        // 3. Select Interest Checkbox (Required field for Form Validation)
        driver.findElement(By.xpath("//input[@data-testid='interest-automation']")).click();
        // 4. Select Single Dropdown (Country)
        WebElement countryDropdown = driver.findElement(By.xpath("//select[@data-testid='country-select']"));
        Select countrySelect = new Select(countryDropdown);
        countrySelect.selectByVisibleText(country);
        // 5. Select Multi-Dropdown (Automation Tool)
        WebElement toolsDropdown = driver.findElement(By.xpath("//select[@data-testid='tools-multi-select']"));
        Select toolsSelect = new Select(toolsDropdown);
        toolsSelect.deselectAll();
        toolsSelect.selectByValue(tool);
        // 6. Fill Textarea (Bio)
        driver.findElement(By.xpath("//textarea[@data-testid='bio']")).sendKeys(bio);
        // 7. Submit the Form
        System.out.println("Submitting form...");
        driver.findElement(By.xpath("//button[@data-testid='submit-btn']")).click();
        // 8. Assert success messages and inputs
        WebElement successMsg = wait.until(
            ExpectedConditions.visibilityOfElementLocated(By.xpath("//div[@data-testid='success-msg']"))
        );
        Assert.assertEquals(successMsg.getText(), "Form submitted successfully!", "Form submission failed!");
        Assert.assertEquals(driver.findElement(By.xpath("//span[@data-testid='result-name']")).getText(), name);
        Assert.assertEquals(driver.findElement(By.xpath("//span[@data-testid='result-email']")).getText(), email);
        Assert.assertEquals(driver.findElement(By.xpath("//span[@data-testid='result-gender']")).getText(), gender);
        Assert.assertEquals(driver.findElement(By.xpath("//span[@data-testid='result-country']")).getText(), country);
        System.out.println("Iteration validated successfully for user: " + name);
    }
    @AfterMethod
    public void tearDown() {
        if (driver != null) {
            System.out.println("Quitting driver session...");
            driver.quit();
        }
    }
}
```

---

## 💻 Test Execution Logs

Below is the clean console output from running the data-driven test case:

```bash
[INFO] Running com.mycodeyatra.tests.DataDrivenTest
Loading data-driven inputs from Excel path: D:\MyCodeYatra\AILearning2026\Repository\mcyt-sel-java\src\test\resources\test_data.xlsx
Executing test iteration for user: John Doe (john.doe@example.com) on target URL: https://practice.mycodeyatra.com/#/form-practice
Selecting gender: Male using locator data-testid=gender-male
Submitting form...
Iteration validated successfully for user: John Doe
Quitting driver session...
Executing test iteration for user: Alice Smith (alice.smith@example.com) on target URL: https://practice.mycodeyatra.com/#/form-practice
Selecting gender: Female using locator data-testid=gender-female
Submitting form...
Iteration validated successfully for user: Alice Smith
Quitting driver session...
Executing test iteration for user: Bob Johnson (bob.johnson@example.com) on target URL: https://practice.mycodeyatra.com/#/form-practice
Selecting gender: Male using locator data-testid=gender-male
Submitting form...
Iteration validated successfully for user: Bob Johnson
Quitting driver session...
Tests run: 3, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 16.851 sec
```

---

## 📊 External Data Formats: Excel vs JSON vs CSV

Automation testing supports multiple data adapters depending on formatting needs:

| Format Type | Read Library | Key Advantage | Disadvantage |
| :--- | :--- | :--- | :--- |
| **Excel (`.xlsx`)** | Apache POI | Business-friendly. Non-technical QA/Product owners can edit rows directly. | Complex API parsing; binary merge conflicts. |
| **JSON (`.json`)** | Jackson / Gson | Supports complex nested objects, arrays, and hierarchies. | Harder for non-programmers to write or edit values. |
| **CSV (`.csv`)** | OpenCSV / Native | Extremely lightweight, fast parsing, and clean git diffs. | Flat data structures only (no multiple sheets or formulas). |

---

## ⚠️ Common Pitfalls

* **Leaking File Streams**: Reading binary Excel workbooks consumes system memory. If files aren't closed cleanly inside helper methods, file locks will crash subsequent JVM runs. Always wrap FileInputStream and XSSFWorkbook resources in **try-with-resources** blocks for auto-closing.
* **Null Cells Exception**: If some cells inside a row are blank, calling `.toString()` on them throws a `NullPointerException`. Utilize POI's `DataFormatter.formatCellValue(cell)` which safely handles empty cells, converting them to blank Strings (`""`) instead of throwing exceptions.
* **Cell Types Mismatches**: Phone numbers or numeric IDs inside Excel cells get parsed by default as decimals (e.g. `9876543210` becomes `9.87654321E9`). Set cell types explicitly to "Text" in Excel or format them programmatically as Strings before returning arrays.

---

## 🙋 Frequently Asked Questions

> **Q: Can we run Data-Driven tests in parallel using TestNG?**
>
> **A:** Yes! Set the `parallel` attribute inside the `@DataProvider` annotation:
> `@DataProvider(name = "data", parallel = true)`. TestNG will automatically execute the test iterations concurrently across thread pools.

> **Q: How do we choose between TestNG `@Parameters` and `@DataProvider`?**
>
> **A:** Use `@Parameters` (via `testng.xml`) for environment configurations (like switching URL, browser, or platform). Use `@DataProvider` for feeding complex data values (like multiple user credentials or form inputs) directly within your test code.
