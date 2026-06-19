---
title: Spreadsheets as Code: Parsing and Asserting Excel Files with Apache POI
date: 08-Aug-2026
lastUpdated: 08-Aug-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [java, selenium, excel-validation, apache-poi, file-downloads, test-automation, backend-validation]
category: Selenium Java
categories: [Selenium Java, Enterprise Validation]
excerpt: >-
  Enterprise systems run on Excel. Learn how to use Apache POI to natively open downloaded .xlsx spreadsheets in memory, navigate Workbooks, and assert Row and Cell data directly from your tests.
readTime: 6 min read
---

# Spreadsheets as Code: Parsing and Asserting Excel Files with Apache POI

In the enterprise world, businesses run on Excel. Financial reports, employee rosters, bulk product uploads, and inventory manifests are almost universally exported as `.xlsx` files.

In our previous tutorial, we learned how to download and parse PDFs using Apache PDFBox. However, PDFBox is completely useless for Excel spreadsheets. 

If your application allows users to download a "Monthly Sales Report" in Excel format, you cannot just verify that the file downloaded. You must open the file programmatically, navigate to specific Rows and Columns, and assert that the mathematical calculations inside the spreadsheet are correct!

In this tutorial, we will learn how to integrate **Apache POI** into our Selenium framework to achieve total mastery over Microsoft Excel files.

---

## 1. Introducing Apache POI

Apache POI (Poor Obfuscation Implementation) is an incredibly powerful Java library designed specifically for reading and writing Microsoft Office formats.

Add both `poi` and `poi-ooxml` (for modern `.xlsx` support) to your `pom.xml`:

```xml
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi</artifactId>
    <version>5.2.4</version>
</dependency>
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi-ooxml</artifactId>
    <version>5.2.4</version>
</dependency>
```

---

## 2. Understanding the Apache POI Architecture

To parse an Excel file, you must drill down through the POI object hierarchy, which perfectly mirrors the structure of a real Excel file:

1. **`XSSFWorkbook`**: Represents the entire `.xlsx` File.
2. **`XSSFSheet`**: Represents a specific Tab/Sheet at the bottom of the workbook.
3. **`XSSFRow`**: Represents a horizontal Row (0-indexed).
4. **`XSSFCell`**: Represents a specific Cell within that Row.

*(Note: If you are working with legacy `.xls` files from 1997-2003, you use the `HSSF` prefix instead of `XSSF`).*

---

## 3. Writing the Excel Validation Test

Let's write a Selenium test that logs into an admin dashboard and downloads a "Q3 Revenue Report". 

We will use POI to open the downloaded spreadsheet in memory, navigate to the "Financials" sheet, loop through the rows to find a specific transaction, and assert the dollar amount!

```java
import org.apache.poi.xssf.usermodel.XSSFCell;
import org.apache.poi.xssf.usermodel.XSSFRow;
import org.apache.poi.xssf.usermodel.XSSFSheet;
import org.apache.poi.xssf.usermodel.XSSFWorkbook;
import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.testng.Assert;
import org.testng.annotations.AfterMethod;
import org.testng.annotations.BeforeMethod;
import org.testng.annotations.Test;
import java.io.File;
import java.io.FileInputStream;
import java.io.IOException;
public class ExcelValidationTest {
    WebDriver driver;
    // Assuming Chrome is configured to download files silently here (like in our PDF tutorial)
    String downloadDir = System.getProperty("user.dir") + "/target/downloads";
    @BeforeMethod
    public void setup() {
        driver = new ChromeDriver();
        // Setup custom ChromeOptions for silent downloads...
    }
    @Test
    public void testFinancialReportExcelData() throws Exception {
        String targetTransactionId = "TXN-5544";
        String expectedRevenue = "5000.00";
        // --- PART 1: The UI Workflow ---
        driver.get("https://practice.mycodeyatra.com/admin/reports");
        driver.findElement(By.id("export-q3-revenue-btn")).click();
        // Custom wait for download to finish
        Thread.sleep(3000); 
        // --- PART 2: The Apache POI Validation ---
        File downloadedExcel = new File(downloadDir + "/Q3_Revenue.xlsx");
        Assert.assertTrue(downloadedExcel.exists(), "The Excel file failed to download!");
        // 1. Open a Data Input Stream to read the binary file
        try (FileInputStream fis = new FileInputStream(downloadedExcel);
             // 2. Load the file into an XSSFWorkbook
             XSSFWorkbook workbook = new XSSFWorkbook(fis)) {
            // 3. Navigate to the specific Sheet by name
            XSSFSheet sheet = workbook.getSheet("Financials");
            Assert.assertNotNull(sheet, "The 'Financials' tab is missing from the Excel file!");
            boolean transactionFound = false;
            // 4. Iterate through all the rows in the sheet
            int rowCount = sheet.getPhysicalNumberOfRows();
            for (int i = 1; i < rowCount; i++) { // Start at 1 to skip the Header row
                XSSFRow row = sheet.getRow(i);
                if (row == null) continue;
                // 5. Extract specific cells (Assuming ID is Column A [0] and Revenue is Column C [2])
                XSSFCell idCell = row.getCell(0);
                XSSFCell revenueCell = row.getCell(2);
                if (idCell != null && idCell.getStringCellValue().equals(targetTransactionId)) {
                    transactionFound = true;
                    // Note: Numeric cells must be retrieved using getNumericCellValue()
                    double actualRevenue = revenueCell.getNumericCellValue();
                    System.out.println("Found Transaction! Extracted Revenue: $" + actualRevenue);
                    // 6. Assert the data!
                    Assert.assertEquals(String.valueOf(actualRevenue), expectedRevenue, 
                        "Revenue mismatch in the exported Excel file!");
                    break;
                }
            }
            Assert.assertTrue(transactionFound, "The target transaction was not found in the spreadsheet!");
        } catch (IOException e) {
            Assert.fail("Failed to parse the Excel document: " + e.getMessage());
        } finally {
            downloadedExcel.delete();
        }
    }
    @AfterMethod
    public void teardown() {
        driver.quit();
    }
}
```

---

## 4. Using POI for Data-Driven Testing (DDT)

While this tutorial focused on *validating* downloaded Excel files, Apache POI is a two-way street.

You can use the exact same logic to *read* a local Excel file before a test starts, and use the rows to drive your automation! If you have an Excel sheet with 50 rows of usernames and passwords, you can write a TestNG `@DataProvider` that uses POI to loop through the sheet and execute the exact same Selenium login script 50 times with different data!

## Conclusion

Exporting massive datasets into `.xlsx` format is a core feature of almost every enterprise application. By integrating Apache POI, you ensure that the raw data living in your databases is being correctly translated, calculated, and formatted into these critical business documents.

We have now validated Emails, PDFs, and Excel spreadsheets. In our final tutorial of this series, we will zoom out and discuss **Enterprise Validation Strategy**, covering how to integrate all of these disconnected backend tools into a cohesive CI/CD pipeline!
