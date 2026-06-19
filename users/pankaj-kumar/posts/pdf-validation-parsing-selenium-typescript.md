---
title: Parsing Binaries: Automated PDF Validation
date: 24-Jan-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, pdf-validation, file-downloads, pdf-parse, enterprise-validation]
category: Selenium TypeScript
categories: [Selenium TypeScript, Enterprise Validation]
excerpt: >-
  Master file downloads in headless Chrome. Learn how to configure Chrome Preferences to download files silently, and use pdf-parse to extract and assert text from binary PDF invoices.
readTime: 4 min read
---

# Parsing Binaries: Automated PDF Validation

We have validated relational databases, NoSQL documents, GraphQL APIs, Kafka event streams, and intercepted SMTP emails. 

Now, we face one of the most notoriously annoying challenges in test automation: **File Downloads and Binary Parsing.**

Imagine your application allows a user to download a monthly invoice. You click the "Download Invoice" button, and a `.pdf` file drops into your local `Downloads` folder. How do you verify that the "Total Amount" on that PDF is exactly `$499.99`? 

You cannot use standard Selenium assertions because a PDF is not HTML. It is a compiled binary file.

Today, we will learn how to configure Chrome to download files into a specific test directory, and how to use Node.js to rip open the PDF and assert its contents.

---

## 1. Configuring Chrome for Headless Downloads

By default, Chrome downloads files to your OS's `Downloads` folder. In a CI/CD pipeline (like GitHub Actions), this is unpredictable. We need to force Chrome to download files into a dedicated directory inside our project workspace.

We achieve this by setting Chrome Preferences in our `WebDriver` setup:

```typescript
import { Builder, Capabilities } from 'selenium-webdriver';
import chrome from 'selenium-webdriver/chrome';
import path from 'path';
import fs from 'fs';
// 1. Define an absolute path to a temporary download folder
const downloadDir = path.resolve(__dirname, '../../temp_downloads');
// Ensure the directory exists
if (!fs.existsSync(downloadDir)) {
  fs.mkdirSync(downloadDir, { recursive: true });
}
// 2. Set Chrome Preferences
const chromeOptions = new chrome.Options();
chromeOptions.setUserPreferences({
  'download.default_directory': downloadDir,
  'download.prompt_for_download': false, // Disable the "Save As" popup
  'plugins.always_open_pdf_externally': true // Force download instead of opening in browser
});
// If running headless, you must explicitly allow downloads
chromeOptions.addArguments('--headless');
// Warning: In newer Chrome versions, headless downloads require this specific flag:
chromeOptions.addArguments('--disable-dev-shm-usage');
export async function createDriver() {
  return await new Builder()
    .forBrowser('chrome')
    .setChromeOptions(chromeOptions)
    .build();
}
```

---

## 2. Installing the PDF Parser

To extract text from a binary PDF file in Node.js, we will use the highly popular `pdf-parse` library.

```bash
npm install pdf-parse
npm install --save-dev @types/pdf-parse
```

---

## 3. Creating the PDF Utility Class

We need a utility function that waits for a file to appear in the download directory (since downloads are asynchronous) and then parses it.

Create `PdfManager.ts`:

```typescript
import fs from 'fs';
import path from 'path';
import pdf from 'pdf-parse';
export class PdfManager {
  /**
   * Waits for a file to exist in the given directory.
   */
  static async waitForFile(filePath: string, timeoutMs: number = 10000): Promise<boolean> {
    const startTime = Date.now();
    while (Date.now() - startTime < timeoutMs) {
      if (fs.existsSync(filePath)) {
        // Wait a tiny bit longer to ensure the file is completely written to disk
        await new Promise(res => setTimeout(res, 500));
        return true;
      }
      await new Promise(res => setTimeout(res, 500));
    }
    throw new Error(`File download timed out: ${filePath}`);
  }
  /**
   * Reads a PDF file from disk and extracts all text content as a massive string.
   */
  static async extractText(filePath: string): Promise<string> {
    const dataBuffer = fs.readFileSync(filePath);
    const parsedData = await pdf(dataBuffer);
    // parsedData.text contains the raw text extracted from all pages!
    return parsedData.text; 
  }
}
```

---

## 4. The E2E Invoice Validation Test

Now we combine our customized Chrome driver with our `PdfManager`. We click the download button, wait for the file to hit our hard drive, parse it, and assert the business logic!

```typescript
import { WebDriver } from 'selenium-webdriver';
import { expect } from 'chai';
import path from 'path';
import fs from 'fs';
import { createDriver } from '../utils/DriverSetup';
import { InvoicePage } from '../pages/InvoicePage';
import { PdfManager } from '../utils/PdfManager';
describe('PDF Invoice Validation', function () {
  let driver: WebDriver;
  const downloadDir = path.resolve(__dirname, '../../temp_downloads');
  const expectedFileName = 'invoice_INV-1002.pdf';
  const filePath = path.join(downloadDir, expectedFileName);
  before(async function () {
    driver = await createDriver(); // Uses our custom Chrome preferences!
  });
  after(async function () {
    await driver.quit();
    // Teardown: Delete the downloaded PDF to keep the workspace clean
    if (fs.existsSync(filePath)) {
      fs.unlinkSync(filePath);
    }
  });
  it('should download the invoice and verify the total amount in the PDF', async function () {
    const invoicePage = new InvoicePage(driver);
    await invoicePage.navigate('INV-1002');
    // 1. UI Layer Action: Click the download button
    await invoicePage.clickDownloadPdf();
    // 2. File System Wait: Wait for the binary file to hit the disk
    console.log(`Waiting for ${expectedFileName} to download...`);
    await PdfManager.waitForFile(filePath);
    // 3. Binary Parsing: Extract the text from the PDF
    const pdfText = await PdfManager.extractText(filePath);
    console.log('PDF Text Extracted Successfully.');
    // 4. Assertion Layer
    expect(pdfText).to.include('Invoice Number: INV-1002');
    expect(pdfText).to.include('Customer: John Doe');
    expect(pdfText).to.include('Total Amount: $499.99');
    expect(pdfText).to.include('Status: PAID');
  });
});
```

## Conclusion

By mastering Chrome Preferences and the `pdf-parse` library, you have eliminated another massive manual testing bottleneck. You can now verify legal documents, tax receipts, and enterprise reports with complete confidence.

But what if the enterprise report isn't a PDF? What if it's a massive spreadsheet containing thousands of rows of financial data?

In our next tutorial, we tackle the final file format in our Enterprise Validation series: **Excel Validation** using `exceljs`!
