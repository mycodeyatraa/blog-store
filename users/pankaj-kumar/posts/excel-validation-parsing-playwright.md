---
title: Spreadsheet Validation & Data-Driven Testing in Playwright
date: 04-May-2025
lastUpdated: 04-May-2025
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: ["playwright", "typescript", "excel", "downloads", "parsing", "data-driven"]
category: Enterprise Validation
categories: ["Enterprise Validation", "UI Automation", "Playwright", "TypeScript", "File Testing"]
excerpt: >-
  Master intercepting Excel UI downloads and parsing `.xlsx` files into JSON arrays using SheetJS for native spreadsheet validation and dynamic Data-Driven tests.
readTime: 4 min read
---

A common requirement in enterprise platforms—such as CRM dashboards or financial portals—is the ability to export tabular data to a `.xlsx` or `.csv` spreadsheet.

Validating that the exported file contains the exact data displayed on the screen is crucial. Since Playwright runs in a Node.js context, we can natively download the file and parse it using libraries like **SheetJS (`xlsx`)** to assert the data mathematically.

### 1. Installation

Install the `xlsx` library. This is the industry standard Node.js library for reading, parsing, and writing Excel files.

```bash
npm install xlsx
```

### 2. Validating Exported Spreadsheets

When a user clicks "Export to Excel", we can intercept the download, save it temporarily, and then parse the sheet into a JSON object. Converting an Excel sheet to JSON makes assertions incredibly simple!

**File:** `tests/excel.spec.ts`

```typescript
import { test, expect } from '@playwright/test';
import * as fs from 'fs';
import * as path from 'path';
import * as xlsx from 'xlsx';
 
test('Download and parse an Exported Excel Report', async ({ page }) => {
  // 1. Setup the Download Listener BEFORE clicking the export button
  const downloadPromise = page.waitForEvent('download');
  
  // 2. Trigger the Excel export in the UI
  await page.goto('/admin/reports');
  await page.click('#export-to-excel-btn');
 
  // 3. Await the download to complete
  const download = await downloadPromise;
  const tempPath = path.join(__dirname, 'temp-report.xlsx');
  
  // 4. Save the file to the local disk
  await download.saveAs(tempPath);
 
  // 5. Read the Excel file natively using SheetJS
  const workbook = xlsx.readFile(tempPath);
  
  // 6. Access the first sheet in the workbook
  const sheetName = workbook.SheetNames[0];
  const worksheet = workbook.Sheets[sheetName];
 
  // 7. Convert the sheet data into a JSON array!
  // This turns rows into objects keyed by the column headers.
  const jsonData: any[] = xlsx.utils.sheet_to_json(worksheet);
 
  // 8. Assert against the structured JSON data natively
  expect(jsonData.length).toBeGreaterThan(0);
  
  // Verify the first row contains the expected export data
  const firstRow = jsonData[0];
  expect(firstRow).toHaveProperty('Transaction ID');
  expect(firstRow['Status']).toBe('COMPLETED');
  expect(firstRow['Amount']).toBe(150.50);
 
  // 9. Always clean up temporary files to prevent disk bloat
  fs.unlinkSync(tempPath);
});
```

### 3. Data-Driven Testing (DDT) using Excel

Parsing Excel isn't just for validating downloads. It is also the perfect way to implement **Data-Driven Testing**. 

Instead of hardcoding test data or using massive JSON files, you can maintain your test cases in a clean Excel spreadsheet, parse it at runtime, and dynamically generate tests!

```typescript
// Outside the test block, read the Excel file
const testDataPath = path.join(__dirname, '../data/test-users.xlsx');
const workbook = xlsx.readFile(testDataPath);
const users = xlsx.utils.sheet_to_json(workbook.Sheets['Users']);
 
// Dynamically generate a test for each row in the Excel sheet!
for (const user of users) {
  test(`Register User: ${user['First Name']}`, async ({ page }) => {
    await page.goto('/register');
    await page.fill('#firstName', user['First Name']);
    await page.fill('#email', user['Email Address']);
    await page.click('#submit');
    
    await expect(page.locator('.success')).toBeVisible();
  });
}
```

### Summary

Integrating `xlsx` into Playwright opens up two massive Enterprise capabilities:
1. Validating that UI export features generate accurate data.
2. Utilizing spreadsheets to power scalable Data-Driven test suites that are easy for non-engineers to maintain!
