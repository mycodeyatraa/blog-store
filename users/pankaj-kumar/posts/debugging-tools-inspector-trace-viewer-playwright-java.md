---
title: Debugging Tools - Playwright Java Foundations
date: 10-Jan-2026
lastUpdated: 10-Jan-2026
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
  Master Playwright Inspector, page.pause(), and Trace Viewer trace.zip artifact analysis for fast post-mortem debugging.
readTime: 9 min read
---

# Debugging Tools - Playwright Java Foundations

Debugging test failures in automated CI/CD pipelines can be time-consuming. Playwright Java offers industry-leading debugging utilities: **Playwright Inspector** for real-time step-through execution and **Trace Viewer** for full post-mortem analysis.

---

## 1. Playwright Inspector & `page.pause()`

To pause test execution and launch the interactive GUI inspector:

```java
// Pauses test execution and opens Playwright Inspector GUI
page.pause();
```

You can also launch Inspector mode from terminal:
```bash
PWDEBUG=1 mvn test -Dtest=FirstE2ETest
```

---

## 2. Generating & Viewing Trace Archives (`trace.zip`)

Trace Viewer captures DOM snapshots, action logs, network traffic, and screenshots for every step of test execution.

```java
package com.mycodeyatra.tests;
 
import com.microsoft.playwright.*;
import java.nio.file.Paths;
import org.junit.jupiter.api.*;
 
public class DebuggingTest {
    @Test
    void testWithTracing() {
        try (Playwright pw = Playwright.create()) {
            Browser b = pw.chromium().launch();
            BrowserContext context = b.newContext();
 
            // 1. Start Tracing before test actions
            context.tracing().start(new Tracing.StartOptions()
                .setScreenshots(true)
                .setSnapshots(true)
                .setSources(true));
 
            Page page = context.newPage();
            page.navigate("https://practice.mycodeyatra.com/sandbox");
 
            // 2. Stop Tracing and export trace.zip file
            context.tracing().stop(new Tracing.StopOptions()
                .setPath(Paths.get("target/trace.zip")));
 
            context.close();
            b.close();
        }
    }
}
```

---

## 3. Opening Trace Files

To inspect a recorded trace file:

```bash
mvn exec:java -e -Dexec.mainClass=com.microsoft.playwright.CLI -Dexec.args="show-trace target/trace.zip"
```

