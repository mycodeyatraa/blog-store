---
title: File Upload - Playwright Java Core UI
date: 15-Jan-2026
lastUpdated: 15-Jan-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [playwright, java, junit5, automation, ui-automation, mycodeyatra]
category: Playwright Java Core UI
categories: [Playwright Java Core UI, Playwright Java, Test Automation]
excerpt: >-
  Automate single and multi-file uploads using page.setInputFiles() without opening native OS file choosers.
readTime: 8 min read
---

# File Upload - Playwright Java Core UI

Automating file uploads in legacy frameworks often required OS-level automation tools (AutoIt, Robot class) because clicking `<input type="file">` opens a native operating system file chooser dialog.

Playwright Java bypasses OS dialogs completely using `page.setInputFiles()`.

---

## 1. File Upload Patterns

```java
// Single File Upload
page.setInputFiles("#file-input", Paths.get("upload/test-document.pdf"));
 
// Multiple Files Upload
page.setInputFiles("#file-input", new Path[] {
    Paths.get("upload/doc1.pdf"),
    Paths.get("upload/doc2.pdf")
});
 
// Clear Selected Files
page.setInputFiles("#file-input", new Path[0]);
```

---

## 2. Production Page Object (`src/main/java/com/mycodeyatra/pages/FileUploadPage.java`)

```java
package com.mycodeyatra.pages;
 
import com.microsoft.playwright.Page;
import com.microsoft.playwright.Locator;
import java.nio.file.Path;
import java.nio.file.Paths;
 
public class FileUploadPage {
    private final Page page;
    private final Locator fileInput;
    private final Locator uploadBtn;
    private final Locator successMsg;
 
    public FileUploadPage(Page page) {
        this.page = page;
        this.fileInput = page.locator("#file-upload-input");
        this.uploadBtn = page.locator("#upload-submit");
        this.successMsg = page.locator(".upload-success");
    }
 
    public void navigateToUploadPage() {
        page.navigate("https://practice.mycodeyatra.com/file-upload-download");
    }
 
    public void uploadFile(String relativeFilePath) {
        fileInput.setInputFiles(Paths.get(relativeFilePath));
        uploadBtn.click();
    }
 
    public Locator getSuccessMessage() {
        return successMsg;
    }
}
```

---

## 3. Summary

Using `setInputFiles()` works seamlessly across headless Linux CI environments, Docker containers, and cloud grid providers without native window dependencies.

