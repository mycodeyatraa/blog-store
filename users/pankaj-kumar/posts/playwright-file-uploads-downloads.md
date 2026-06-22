---
title: "Handling File Uploads and Downloads"
date: "2025-03-03"
description: "Testing file systems can be tricky. Learn how Playwright intercepts downloads and handles file uploads without triggering native OS file dialogs."
tags: ["Playwright", "TypeScript", "Upload", "Download", "Files"]
---

Welcome to Blog 19 of the **Playwright TypeScript Mastery Series**!

One of the oldest rules of Web Automation is: **You cannot automate the Operating System.** 

When you click a "Choose File" button, Windows or macOS opens a native file explorer dialog. A web automation tool like Selenium or Playwright has absolutely no control over that OS window. So how do we upload or download files?

We bypass the OS completely!

### 1. Uploading Files

Instead of clicking the "Upload" button, we directly inject the file path into the hidden HTML `<input type="file">` tag.

Let's test this on our live sandbox (`https://practice.mycodeyatra.com/#/upload-download`):

Create `tests/blog19_upload_download.spec.ts`:

```typescript
import { test, expect } from '@playwright/test';
import * as fs from 'fs';
import * as path from 'path';
 
test('Uploading a file', async ({ page }) => {
  await page.goto('https://practice.mycodeyatra.com/#/upload-download');
  
  // 1. Create a dummy file locally to upload
  const filePath = path.join(__dirname, 'test-upload.txt');
  fs.writeFileSync(filePath, 'Hello, Playwright Upload!');
  
  // 2. We don't click the "Select File" button! 
  // Instead, we directly target the <input type="file"> tag:
  await page.getByTestId('file-upload-input').setInputFiles(filePath);
  
  // 3. Verify the upload status updated
  const status = page.getByTestId('upload-status');
  await expect(status).toContainText('Successfully uploaded: test-upload.txt');
  
  // Clean up
  fs.unlinkSync(filePath);
});
```

### 2. Downloading Files

Just like handling new tabs (`waitForEvent('page')`), we must tell Playwright to wait for a download event *before* we click the download button.

Playwright will download the file to a temporary invisible folder. We can then save it to a known location, read its contents, and assert the file is correct!

```typescript
test('Downloading a file', async ({ page }) => {
  await page.goto('https://practice.mycodeyatra.com/#/upload-download');
  
  // Set custom text to download
  const expectedText = 'Playwright downloaded this!';
  await page.getByTestId('download-text-input').fill(expectedText);
  
  // 1. We must wait for the "download" event BEFORE clicking the download button!
  const [download] = await Promise.all([
    page.waitForEvent('download'),
    page.getByTestId('download-btn').click()
  ]);
  
  // 2. Save the temporary file to a specific path
  const downloadPath = path.join(__dirname, download.suggestedFilename());
  await download.saveAs(downloadPath);
  
  // 3. Read the actual file from our computer and verify its contents!
  const fileContent = fs.readFileSync(downloadPath, 'utf8');
  expect(fileContent).toBe(expectedText);
  
  // Clean up
  fs.unlinkSync(downloadPath);
  console.log('Successfully downloaded and verified the file contents!');
});
```

### Execution Output

When you run `npx playwright test tests/blog19_upload_download.spec.ts`:

```
Running 2 tests using 1 worker
 
  OK   1 tests/blog19_upload_download.spec.ts:7:7 > Blog 19: Handling File Uploads and Downloads > Uploading a file (631ms)
Successfully downloaded and verified the file contents!
  OK   2 tests/blog19_upload_download.spec.ts:27:7 > Blog 19: Handling File Uploads and Downloads > Downloading a file (857ms)
 
  2 passed (2.9s)
```

### Conclusion

File handling is intimidating at first, but Playwright's `setInputFiles()` and `waitForEvent('download')` APIs make it robust and OS-independent.

In **Blog 20**, we will cover the final primary category of user interactions: **Advanced Mouse Actions** (Hovering, Drag and Drop, and Context Menus)!
