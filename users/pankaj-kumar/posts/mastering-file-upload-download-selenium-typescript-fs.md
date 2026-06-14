---
title: Mastering File Upload & Download in Selenium TypeScript
date: 11-Nov-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [typescript, selenium, file-upload, file-download, fs-module]
category: Selenium TypeScript
categories: [Selenium TypeScript, UI Automation, Core Automation]
excerpt: >-
  Native OS file dialogs will freeze your Selenium tests. Learn how to bypass them to automate File Uploads, and use Node's fs module to verify File Downloads.
readTime: 6 min read
---

# Mastering File Upload & Download in Selenium TypeScript

Have you ever struggled to automate file uploads, or verify that a downloaded file actually contains the correct data?

Unlike standard web elements, the native Operating System file explorer dialogs cannot be automated using Selenium. If you click the "Browse File" button, your browser will spawn a native Windows/Mac dialog, and Selenium will freeze indefinitely waiting for it to close!

In this tutorial, we will learn how to bypass the native dialog entirely using the hidden `<input type="file">` element, and how to verify downloaded files using Node.js modules on **[practice.mycodeyatra.com/#/upload-download](https://practice.mycodeyatra.com/#/upload-download)**.

---

## 1. File Uploads in Selenium

The secret to uploading files in Selenium is that you **never click the Upload button**. 

Instead, you locate the `<input type="file">` element in the HTML DOM (which is often hidden via CSS `display: none;`) and send the **absolute path** of your local file directly to it using `.sendKeys()`.

```typescript
// 1. Locate the file input element
const uploadInput = await driver.findElement(By.css("[data-testid='file-upload-input']"));
// 2. Send the absolute path of the local file
await uploadInput.sendKeys("C:\\path\\to\\your\\file.txt");
```

---

## 2. File Downloads & Verification

When you click a download button, the browser automatically saves the file to your default Downloads directory (usually `C:\Users\Username\Downloads`).

Since downloading takes time depending on network speed, we cannot instantly assert the file exists. We must write a custom polling loop using the Node.js `fs` (File System) module to check if the file has arrived, and then read its contents.

```typescript
import * as fs from "fs";
import * as path from "path";
import * as os from "os";
// Find default downloads directory
const downloadDir = path.join(os.homedir(), "Downloads");
const downloadedFilePath = path.join(downloadDir, "dynamic_download.txt");
// Wait for the file to exist (Polling)
let fileExists = false;
for (let i = 0; i < 10; i++) {
  if (fs.existsSync(downloadedFilePath)) {
    fileExists = true;
    break;
  }
  await new Promise(resolve => setTimeout(resolve, 1000));
}
```

---

## 3. Writing the Upload & Download Test

Let's write a comprehensive Jest test. We will use `fs.writeFileSync()` to dynamically create a dummy file on the fly, upload it, download a new file, and verify its contents!

Create `tests/upload_download.test.ts`:

```typescript
import { Builder, WebDriver, By, until } from "selenium-webdriver";
import "chromedriver";
import * as path from "path";
import * as fs from "fs";
import * as os from "os";
describe("Core UI Automation - File Upload and Download", () => {
  let driver: WebDriver;
  const downloadDir = path.join(os.homedir(), "Downloads");
  beforeAll(async () => {
    driver = await new Builder().forBrowser("chrome").build();
    await driver.manage().window().maximize();
    await driver.manage().setTimeouts({ implicit: 5000 });
  });
  afterAll(async () => {
    if (driver) {
      await driver.quit();
    }
  });
  it("Should upload a local file", async () => {
    console.log("Navigating to Upload/Download Page...");
    await driver.get("https://practice.mycodeyatra.com/#/upload-download");
    // Wait for page to render
    await driver.wait(until.elementLocated(By.css("[data-testid='file-upload-input']")), 5000);
    // Create a dummy file to upload
    const filePath = path.join(__dirname, "dummy_upload.txt");
    fs.writeFileSync(filePath, "This is a test upload file.");
    // Upload the file by sending absolute path
    const uploadInput = await driver.findElement(By.css("[data-testid='file-upload-input']"));
    await uploadInput.sendKeys(filePath);
    console.log(`Sent file path: ${filePath}`);
    // Verify upload success
    const statusMsg = await driver.findElement(By.css("[data-testid='upload-status']"));
    expect(await statusMsg.getText()).toContain("Successfully uploaded: dummy_upload.txt");
    console.log("Verified upload success message!");
    // Clean up
    fs.unlinkSync(filePath);
  });
  it("Should download a file and verify its contents", async () => {
    const customText = "Hello from TypeScript Automation!";
    const textArea = await driver.findElement(By.css("[data-testid='download-text-input']"));
    await textArea.clear();
    await textArea.sendKeys(customText);
    const downloadBtn = await driver.findElement(By.css("[data-testid='download-btn']"));
    await downloadBtn.click();
    console.log("Clicked Download Button");
    // Wait for the file to be downloaded (Polling)
    const downloadedFilePath = path.join(downloadDir, "dynamic_download.txt");
    let fileExists = false;
    for (let i = 0; i < 10; i++) {
      if (fs.existsSync(downloadedFilePath)) {
        fileExists = true;
        break;
      }
      await new Promise(resolve => setTimeout(resolve, 1000));
    }
    expect(fileExists).toBe(true);
    // Verify file contents
    const fileContent = fs.readFileSync(downloadedFilePath, "utf8");
    expect(fileContent).toEqual(customText);
    console.log("Successfully verified downloaded file contents!");
    // Clean up
    if (fs.existsSync(downloadedFilePath)) {
      fs.unlinkSync(downloadedFilePath);
    }
  });
});
```

---

## 4. Test Execution Output

```text
> mcyt-sel-typescript@1.0.0 test
> jest
 PASS  tests/upload_download.test.ts (8.152 s)
  Core UI Automation - File Upload and Download
    √ Should upload a local file (1841 ms)
    √ Should download a file and verify its contents (2240 ms)
  console.log
    Navigating to Upload/Download Page...
  console.log
    Sent file path: D:\MyCodeYatra\AILearning2026\Repository\mcyt-sel-typescript\tests\dummy_upload.txt
  console.log
    Verified upload success message!
  console.log
    Clicked Download Button
  console.log
    Successfully verified downloaded file contents!
Test Suites: 1 passed, 1 total
Tests:       2 passed, 2 total
Snapshots:   0 total
Time:        8.452 s
Ran all test suites.
```

## Conclusion

File operations require interacting directly with the underlying Operating System. By leveraging Selenium's ability to inject file paths into input elements, and utilizing Node's powerful native `fs` module, you can seamlessly automate complete End-to-End file workflows!

In our next tutorial, we will explore navigating between **Multiple Windows and Browser Tabs**!
