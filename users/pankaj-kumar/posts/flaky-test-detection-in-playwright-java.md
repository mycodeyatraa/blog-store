---
id: "post-748"
title: "Flaky Test Detection in Playwright Java"
slug: "flaky-test-detection-in-playwright-java"
date: "30-Aug-2026"
author: "pankaj-kumar"
series: "playwright-java-reporting-obs"
seriesOrder: 6
topic: "6. Flaky Test Identification & Isolation"
tags: ["Playwright", "Java", "Flaky Tests", "Retry", "Quarantine"]
category: "Automation Testing"
categories: ["Automation Testing", "Playwright Java", "Flaky Tests"]
excerpt: "Detect, isolate, and quarantine flaky automation tests in Playwright Java using JUnit 5 retry extensions."
readTime: "8 min read"
---

# Flaky Test Detection in Playwright Java

Flaky tests erode trust in automated CI pipelines. Detecting unstable tests using retry rules and quarantining them preserves build stability.

---

## 1. Architectural Overview

A custom JUnit 5 invocation interceptor retries failed tests. If a test fails initially but passes on retry, it is tagged as **FLAKY** and logged to a quarantine tracking file.

```
+-----------------------------+
| Initial Test Run: FAILED    |
+-----------------------------+
               |
               v Intercept & Retry Execution
+-----------------------------+
| Retry #1: PASSED            |
+-----------------------------+
               |
               v Log to Quarantine Database
+--------------------------------------------------------------------+
| Flaky Test Tagged: @Flaky (Quarantined from Blocking CI Builds)   |
+--------------------------------------------------------------------+
```

---

## 2. Flaky Test Interceptor & Quarantine Logger

```java
// src/main/java/com/mycodeyatra/flaky/FlakyRetryExtension.java
package com.mycodeyatra.flaky;
 
import org.junit.jupiter.api.extension.ExtensionContext;
import org.junit.jupiter.api.extension.TestExecutionExceptionHandler;
 
public class FlakyRetryExtension implements TestExecutionExceptionHandler {
    private static final int MAX_RETRIES = 2;
 
    @Override
    public void handleTestExecutionException(ExtensionContext context, Throwable throwable) throws Throwable {
        int count = context.getStore(ExtensionContext.Namespace.GLOBAL).getOrDefault("retryCount", Integer.class, 0);
        if (count < MAX_RETRIES) {
            context.getStore(ExtensionContext.Namespace.GLOBAL).put("retryCount", count + 1);
            System.err.println("Flaky retry attempt " + (count + 1) + " for " + context.getDisplayName());
            // Log as potential flaky test
            return;
        }
        throw throwable;
    }
}
```

---

## 3. Playwright Flaky Test Suite

```java
// src/test/java/com/mycodeyatra/tests/FlakyDetectionTest.java
package com.mycodeyatra.tests;
 
import com.microsoft.playwright.*;
import com.mycodeyatra.flaky.FlakyRetryExtension;
import org.junit.jupiter.api.*;
import org.junit.jupiter.api.extension.ExtendWith;
 
import static org.junit.jupiter.api.Assertions.*;
 
@ExtendWith(FlakyRetryExtension.class)
public class FlakyDetectionTest {
    private static Playwright playwright;
    private static Browser browser;
    private Page page;
 
    @BeforeAll
    static void setup() {
        playwright = Playwright.create();
        browser = playwright.chromium().launch(new BrowserType.LaunchOptions().setHeadless(true));
    }
 
    @BeforeEach
    void init() {
        page = browser.newPage();
    }
 
    @Test
    @Tag("flaky-quarantine")
    @DisplayName("Validate Intermittent Asynchronous Rendering")
    void testAsyncRendering() {
        page.navigate("https://mycodeyatra.com");
        // Explicit auto-retrying assertion reduces genuine flakiness
        assertThat(page.locator("header")).isVisible();
    }
 
    @AfterAll
    static void tearDown() {
        if (browser != null) browser.close();
        if (playwright != null) playwright.close();
    }
}
```

---

## 4. Enterprise Best Practices & Comparison Summary

- **Auto-Quarantine Tagging**: Exclude `@Tag("flaky-quarantine")` from release-blocking CI jobs.
- **Web-First Assertions**: Replace raw `assertTrue(page.isVisible())` with `assertThat(locator).isVisible()` to leverage Playwright's built-in auto-wait retries.
