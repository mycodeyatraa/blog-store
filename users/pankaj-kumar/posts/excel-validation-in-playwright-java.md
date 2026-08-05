---
id: "post-740"
title: "Excel Validation in Playwright Java"
slug: "excel-validation-in-playwright-java"
date: "23-Aug-2026"
author: "pankaj-kumar"
series: "playwright-java-enterprise-data"
seriesOrder: 9
topic: "9. Excel Data & Spreadsheet Validation"
tags: ["Playwright", "Java", "Excel", "Apache POI", "Spreadsheet"]
category: "Automation Testing"
categories: ["Automation Testing", "Playwright Java", "Excel Validation"]
excerpt: "Download and parse Excel spreadsheets (.xlsx) using Apache POI in Playwright Java test scripts."
readTime: "8 min read"
---

# Excel Validation in Playwright Java

Data-heavy web applications export tabular reports and financial summaries to Microsoft Excel (.xlsx) files. Validating cell values and headers ensures data integrity.

---

## 1. Architectural Overview

Playwright Java captures spreadsheet download events, saving the Excel file locally. Apache POI inspects workbooks, sheets, rows, and cells for automated verification.

```
+-------------------------------+       +------------------------------------+
| Playwright Download Event     | ----> | Web Application Excel Export       |
+-------------------------------+       +------------------------------------+
                |
                v Save .xlsx file
+----------------------------------------------------------------------------+
| Apache POI Workbook Engine (Sheet Iteration, Cell Value Type Checking)     |
+----------------------------------------------------------------------------+
```

---

## 2. Apache POI Excel Reader Utility

```java
// src/main/java/com/mycodeyatra/excel/ExcelReader.java
package com.mycodeyatra.excel;
 
import org.apache.poi.ss.usermodel.*;
import org.apache.poi.xssf.usermodel.XSSFWorkbook;
 
import java.io.File;
import java.io.FileInputStream;
import java.io.IOException;
 
public class ExcelReader {
 
    public static String getCellValue(File excelFile, String sheetName, int rowIdx, int colIdx) throws IOException {
        try (FileInputStream fis = new FileInputStream(excelFile);
             Workbook workbook = new XSSFWorkbook(fis)) {
            
            Sheet sheet = workbook.getSheet(sheetName);
            Row row = sheet.getRow(rowIdx);
            Cell cell = row.getCell(colIdx);
 
            if (cell == null) return "";
            switch (cell.getCellType()) {
                case STRING: return cell.getStringCellValue();
                case NUMERIC: return String.valueOf(cell.getNumericCellValue());
                case BOOLEAN: return String.valueOf(cell.getBooleanCellValue());
                default: return "";
            }
        }
    }
}
```

---

## 3. Playwright Excel Validation Test

```java
// src/test/java/com/mycodeyatra/tests/ExcelDownloadValidationTest.java
package com.mycodeyatra.tests;
 
import com.microsoft.playwright.*;
import com.mycodeyatra.excel.ExcelReader;
import org.junit.jupiter.api.*;
 
import java.io.File;
import java.io.IOException;
import java.nio.file.Path;
import java.nio.file.Paths;
 
import static org.junit.jupiter.api.Assertions.*;
 
public class ExcelDownloadValidationTest {
    private static Playwright playwright;
    private static Browser browser;
    private Page page;
 
    @BeforeAll
    static void setup() {
        playwright = Playwright.create();
        browser = playwright.chromium().launch(new BrowserType.LaunchOptions().setHeadless(true));
    }
 
    @BeforeEach
    void init() {
        page = browser.newPage();
    }
 
    @Test
    @DisplayName("Download Report Excel and Assert Cell Headers and Data")
    void testExcelReportDownload() throws IOException {
        page.navigate("https://mycodeyatra.com/reports");
 
        // 1. Capture Download Event
        Download download = page.waitForDownload(() -> {
            page.click("#export-excel-btn");
        });
 
        Path destPath = Paths.get("target/downloads/sales_report.xlsx");
        download.saveAs(destPath);
        File excelFile = destPath.toFile();
 
        // 2. Validate Cell Data with Apache POI
        assertTrue(excelFile.exists(), "Excel file must exist");
        
        String header0 = ExcelReader.getCellValue(excelFile, "Sales", 0, 0);
        String header1 = ExcelReader.getCellValue(excelFile, "Sales", 0, 1);
        assertEquals("Transaction ID", header0);
        assertEquals("Amount", header1);
 
        String firstTxn = ExcelReader.getCellValue(excelFile, "Sales", 1, 0);
        assertEquals("TXN-5001", firstTxn);
    }
 
    @AfterAll
    static void tearDown() {
        if (browser != null) browser.close();
        if (playwright != null) playwright.close();
    }
}
```

---

## 4. Enterprise Best Practices & Comparison Summary

- **Resource Disposal**: Always manage `FileInputStream` and `Workbook` inside try-with-resources blocks to prevent memory leaks.
- **Header Mapping**: Map column indices dynamically using header row strings to handle dynamic column reordering.
