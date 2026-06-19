---
title: Parsing Spreadsheets: Automated Excel Validation
date: 25-Jan-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, excel-validation, exceljs, spreadsheet, file-downloads, enterprise-validation]
category: Selenium TypeScript
categories: [Selenium TypeScript, Enterprise Validation]
excerpt: >-
  Automate the 'Export to Excel' feature. Learn how to intercept massive `.xlsx` downloads and use the exceljs library to programmatically assert workbook sheets, rows, and cells.
readTime: 4 min read
---

# Parsing Spreadsheets: Automated Excel Validation

If you work in Enterprise QA—especially in Finance, Healthcare, or Logistics—you will inevitably encounter the "Export to Excel" button.

Users need the ability to download massive reports, manipulate the data in `.xlsx` format, and upload it back into the system. As an Automation Engineer, you must verify that the downloaded spreadsheet contains the exact data displayed on the UI.

In our last tutorial, we configured headless Chrome to download files into a temporary directory. Today, we will use that same download logic, but instead of parsing a PDF, we will rip open an Excel workbook using the powerful `exceljs` library!

---

## 1. Installing ExcelJS

While there are many libraries for parsing spreadsheets (like `xlsx`), `exceljs` is arguably the most modern, Promise-based, and TypeScript-friendly option available in the Node.js ecosystem.

```bash
npm install exceljs
npm install --save-dev @types/exceljs
```

---

## 2. Building the Excel Parsing Utility

We need a utility class that can open an `.xlsx` file from our downloads folder, target a specific worksheet, and allow us to query specific rows and columns.

Create `ExcelManager.ts`:

```typescript
import ExcelJS from 'exceljs';
export class ExcelManager {
  /**
   * Reads an Excel file and returns the specified Worksheet.
   */
  static async getWorksheet(filePath: string, sheetName: string = 'Sheet1'): Promise<ExcelJS.Worksheet> {
    const workbook = new ExcelJS.Workbook();
    await workbook.xlsx.readFile(filePath);
    const worksheet = workbook.getWorksheet(sheetName);
    if (!worksheet) {
      throw new Error(`Worksheet '${sheetName}' not found in ${filePath}`);
    }
    return worksheet;
  }
  /**
   * Searches a column for a specific value and returns the entire Row object.
   */
  static async findRowByColumnValue(
    filePath: string, 
    sheetName: string, 
    searchColIndex: number, 
    searchValue: string
  ): Promise<ExcelJS.Row | null> {
    const worksheet = await this.getWorksheet(filePath, sheetName);
    let targetRow: ExcelJS.Row | null = null;
    // Iterate through every row in the sheet
    worksheet.eachRow((row, rowNumber) => {
      // Skip headers (assuming row 1 is the header)
      if (rowNumber === 1) return; 
      const cellValue = row.getCell(searchColIndex).text;
      if (cellValue === searchValue) {
        targetRow = row;
      }
    });
    return targetRow;
  }
}
```

---

## 3. The E2E Spreadsheet Assertion

Let's assume our application has a "Financial Reports" page. We click "Export Q3 Earnings", wait for the `Q3_Report.xlsx` file to download, and then assert that "Acme Corp" has a revenue of exactly `$1,000,000` listed in the spreadsheet.

```typescript
import { WebDriver } from 'selenium-webdriver';
import { expect } from 'chai';
import path from 'path';
import fs from 'fs';
import { createDriver } from '../utils/DriverSetup';
import { ReportsPage } from '../pages/ReportsPage';
import { PdfManager } from '../utils/PdfManager'; // Reusing our wait logic!
import { ExcelManager } from '../utils/ExcelManager';
describe('Excel Report Validation', function () {
  let driver: WebDriver;
  const downloadDir = path.resolve(__dirname, '../../temp_downloads');
  const expectedFileName = 'Q3_Report.xlsx';
  const filePath = path.join(downloadDir, expectedFileName);
  before(async function () {
    // Uses the Chrome Preferences we setup in the PDF tutorial
    driver = await createDriver(); 
  });
  after(async function () {
    await driver.quit();
    // Teardown
    if (fs.existsSync(filePath)) {
      fs.unlinkSync(filePath);
    }
  });
  it('should export the report and verify row data in Excel', async function () {
    const reportsPage = new ReportsPage(driver);
    await reportsPage.navigate();
    // 1. UI Layer Action: Click Export
    await reportsPage.clickExportQ3Report();
    // 2. Wait for download
    console.log(`Waiting for ${expectedFileName} to download...`);
    await PdfManager.waitForFile(filePath); // Reusing the wait utility
    // 3. Binary Parsing: Find "Acme Corp" in Column 2
    console.log('Searching Excel Workbook for Acme Corp...');
    const acmeRow = await ExcelManager.findRowByColumnValue(filePath, 'Earnings', 2, 'Acme Corp');
    expect(acmeRow).to.not.be.null;
    // 4. Data Assertion: Verify the Revenue in Column 5
    const revenue = acmeRow!.getCell(5).value;
    expect(revenue).to.equal(1000000); // Note: exceljs parses numeric cells as real Numbers!
  });
});
```

## Conclusion

Testing file downloads doesn't have to be a manual chore. By combining headless Chrome download preferences with the `exceljs` library, we can programmatically assert massive datasets inside binary spreadsheets. 

This is the ultimate level of Enterprise QA: validating the UI, the Database, the APIs, the Kafka streams, the Emails, and the Physical File Exports.

In the final tutorial of Phase 11, we will zoom out and discuss how to structure all of these concepts into a cohesive **Enterprise Validation Strategy**.
