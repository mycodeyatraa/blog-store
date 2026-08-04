---
title: File Download - Playwright Java Core UI
date: 16-Jan-2026
lastUpdated: 16-Jan-2026
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
  Intercept file downloads using page.waitForDownload() and verify downloaded files in Java.
readTime: 9 min read
---

# File Download - Playwright Java Core UI

Testing file downloads (PDF receipts, CSV exports, Excel reports) is a frequent requirement in enterprise applications. 

Playwright Java captures downloads as first-class `Download` events without configuring complex browser download preferences or binary MIME types.

---

## 1. The `waitForDownload()` Pattern

```java
// Intercept and capture the download event
Download download = page.waitForDownload(() -> {
    page.click("#download-csv-btn");
});
 
// Access download metadata
System.out.println("Suggested Filename: " + download.suggestedFilename());
 
// Save file to custom target path
download.saveAs(Paths.get("target/downloads/" + download.suggestedFilename()));
```

---

## 2. Production Page Object (`src/main/java/com/mycodeyatra/pages/FileDownloadPage.java`)

```java
package com.mycodeyatra.pages;
 
import com.microsoft.playwright.Page;
import com.microsoft.playwright.Download;
import java.nio.file.Paths;
 
public class FileDownloadPage {
    private final Page page;
 
    public FileDownloadPage(Page page) {
        this.page = page;
    }
 
    public void navigateToDownloadPage() {
        page.navigate("https://practice.mycodeyatra.com/file-upload-download");
    }
 
    public Download downloadSampleFile() {
        return page.waitForDownload(() -> {
            page.click("#download-sample-btn");
        });
    }
}
```

---

## 3. Key Takeaways

1. **Event Listener Wrapper**: Wrap the click action inside `page.waitForDownload(() -> { ... })`.
2. **File Cleanup**: Use `download.delete()` after test verification to prevent test artifact buildup.

