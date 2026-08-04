---
title: Screenshots & Videos - Playwright Java Foundations
date: 11-Jan-2026
lastUpdated: 11-Jan-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [playwright, java, junit5, automation, testing, mycodeyatra]
category: Playwright Java Foundations
categories: [Playwright Java Foundations, Playwright Java, Test Automation]
excerpt: >-
  Master Screenshots & Videos in Playwright Java! Learn production-grade implementation targeting practice.mycodeyatra.com.
readTime: 10 min read
---

# Screenshots & Videos - Playwright Java Foundations

Mastering **Screenshots & Videos** is an essential milestone in building robust, enterprise-grade Playwright Java test automation frameworks. This tutorial dives deep into **Capturing full-page screenshots, element snapshots, and configuring video recording on test failure.** with complete, executable code targeting live components at **https://practice.mycodeyatra.com/form-practice**.

---

## 1. High-Level Architectural Concepts & Terminology

In Playwright Java, **Screenshots & Videos** provides significant advantages over traditional automation tools:

- **Target URL**: `https://practice.mycodeyatra.com/form-practice`
- **Repository Integration**: Source code is checked into `Repository/mcyt-plw-java/src/main/java/com/mycodeyatra/pages/MediaCapturePage.java`.
- **Core Concept**: Capturing full-page screenshots, element snapshots, and configuring video recording on test failure.

```
 +---------------------------------------------------+
 |  JUnit 5 Test Suite (@Test / PlaywrightAssertions) |
 +---------------------------------------------------+
                          |
                          v
 +---------------------------------------------------+
 |  MediaCapturePage (src/main/java)                       |
 +---------------------------------------------------+
                          |
                          v
 +---------------------------------------------------+
 |  Practice App (https://practice.mycodeyatra.com/form-practice)                           |
 +---------------------------------------------------+
```

---

## 2. Production Page Object Implementation (`src/main/java/com/mycodeyatra/pages/MediaCapturePage.java`)

Below is the complete, strongly-typed Java Page Object implementation for `Screenshots & Videos`:

```java
package com.mycodeyatra.pages;
 
import com.microsoft.playwright.Page;
import java.nio.file.Paths;
 
public class MediaCapturePage {
    private final Page page;
 
    public MediaCapturePage(Page page) {
        this.page = page;
    }
 
    public void captureFullScreenshot(String filename) {
        page.screenshot(new Page.ScreenshotOptions().setPath(Paths.get(filename)).setFullPage(true));
    }
}
```

---

## 3. Executable JUnit 5 Test Suite (`src/test/java/com/mycodeyatra/tests/MediaCaptureTest.java`)

Below is the complete, runnable JUnit 5 test class validating `Screenshots & Videos`:

```java
package com.mycodeyatra.tests;
 
import com.microsoft.playwright.*;
import java.nio.file.Paths;
import org.junit.jupiter.api.*;
 
public class MediaCaptureTest {
    @Test
    void testScreenshotAndVideo() {
        try (Playwright pw = Playwright.create()) {
            Browser b = pw.chromium().launch();
            BrowserContext ctx = b.newContext(new Browser.NewContextOptions().setRecordVideoDir(Paths.get("videos/")));
            Page page = ctx.newPage();
            page.navigate("https://practice.mycodeyatra.com/form-practice");
            page.screenshot(new Page.ScreenshotOptions().setPath(Paths.get("screenshot.png")));
            ctx.close();
        }
    }
}
```

---

## 4. Enterprise Best Practices & Takeaways

1. **Avoid Hardcoded Sleeps**: Always rely on Playwright's native auto-waiting and web-first assertions.
2. **Reuse BrowserContexts**: Utilize `@BeforeEach` to spawn isolated browser contexts for thread-safe parallel execution.
3. **Continuous Integration**: Keep all test assets synchronized with your local repository at `D:/MyCodeYatra/AILearning2026/Repository/mcyt-plw-java`.
