---
id: "post-784"
title: "File Upload Automation in Playwright C#"
slug: "file-upload-automation-in-playwright-csharp"
date: "15-Feb-2026"
author: "pankaj-kumar"
series: "playwright-csharp-core-ui"
seriesOrder: 4
topic: "4. File Upload Automation in C#"
tags: ["Playwright", "C#", "File Upload", "FileChooser", "Files"]
category: "Automation Testing"
categories: ["Automation Testing", "Playwright C#", "File Upload"]
excerpt: "Automate single and multiple file uploads in Playwright C# using `SetInputFilesAsync` and `RunAndWaitForFileChooserAsync`."
readTime: "8 min read"
---

# File Upload Automation in Playwright C#

Testing file upload functionality requires handling standard `<input type="file">` elements as well as custom drag-and-drop upload widgets.

---

## 1. File Upload Interaction Options

```
+---------------------------------------------------------------------------------+
| Option A: Direct Input Setter (Page.GetByLabel("Upload").SetInputFilesAsync())  |
+---------------------------------------------------------------------------------+
                                       or
+---------------------------------------------------------------------------------+
| Option B: FileChooser Event Interceptor (Page.RunAndWaitForFileChooserAsync())  |
+---------------------------------------------------------------------------------+
```

---

## 2. Playwright C# File Upload Test

```csharp
// FileUploadTest.cs
using Microsoft.Playwright;
using Microsoft.Playwright.NUnit;
using NUnit.Framework;
using System.IO;
using System.Threading.Tasks;
 
namespace MyCodeYatra.PlaywrightTests;
 
[TestFixture]
public class FileUploadTest : PageTest
{
    [Test]
    public async Task TestSingleFileUpload()
    {
        await Page.GotoAsync("https://mycodeyatra.com/upload");
 
        // Create temporary test file
        string tempFilePath = Path.Combine(Path.GetTempPath(), "test_document.pdf");
        await File.WriteAllTextAsync(tempFilePath, "Sample PDF Content for Upload");
 
        // Option A: Direct input locator file assignment
        await Page.Locator("input[type='file']").SetInputFilesAsync(tempFilePath);
        await Page.GetByRole(AriaRole.Button, new() { Name = "Submit Upload" }).ClickAsync();
 
        await Expect(Page.Locator(".upload-success")).ToBeVisibleAsync();
    }
 
    [Test]
    public async Task TestCustomFileChooserUpload()
    {
        await Page.GotoAsync("https://mycodeyatra.com/upload");
 
        string tempFilePath = Path.Combine(Path.GetTempPath(), "sample_image.png");
        await File.WriteAllTextAsync(tempFilePath, "Dummy Image Data");
 
        // Option B: Intercept file chooser dialog
        var fileChooser = await Page.RunAndWaitForFileChooserAsync(async () =>
        {
            await Page.Locator(".custom-upload-button").ClickAsync();
        });
 
        await fileChooser.SetFilesAsync(tempFilePath);
        await Expect(Page.Locator(".file-name")).ToHaveTextAsync("sample_image.png");
    }
}
```

---

## 3. Enterprise Best Practices & Comparison Summary

- **Clear Temporary Files**: Clean up created test files in `TearDown` blocks to prevent build directory clutter.
- **Multiple Files**: Pass string arrays (`SetInputFilesAsync(new[] { "file1.txt", "file2.txt" })`) to upload multiple files simultaneously.
