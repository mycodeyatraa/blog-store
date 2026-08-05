---
id: "post-745"
title: "Custom Reports in Playwright Java"
slug: "custom-reports-in-playwright-java"
date: "27-Aug-2026"
author: "pankaj-kumar"
series: "playwright-java-reporting-obs"
seriesOrder: 3
topic: "3. Custom HTML & JSON Reporters"
tags: ["Playwright", "Java", "Custom Reporter", "JUnit 5", "HTML"]
category: "Automation Testing"
categories: ["Automation Testing", "Playwright Java", "Custom Reports"]
excerpt: "Build lightweight custom HTML and JSON test reporters for Playwright Java frameworks using custom JUnit 5 extensions."
readTime: "8 min read"
---

# Custom Reports in Playwright Java

When third-party reporting tools are unavailable or enterprise security mandates lightweight local artifacts, building custom HTML/JSON reporters gives total control over summary formatting.

---

## 1. Architectural Overview

Custom reporters implement JUnit 5 `TestExecutionListener`, collecting test names, execution status, duration, and failure trace strings to generate bespoke HTML dashboard pages.

```
+------------------------------------------+
| Playwright Test Suite                    |
+------------------------------------------+
                     |
                     v Listens to test lifecycle events
+------------------------------------------+
| Custom TestExecutionListener (Java)       |
| - testPlanExecutionFinished()            |
| - executionFinished()                    |
+------------------------------------------+
                     |
                     v Generates target/custom-report.html
+------------------------------------------+
| Lightweight Custom HTML / JSON Report    |
+------------------------------------------+
```

---

## 2. Custom HTML Report Engine

```java
// src/main/java/com/mycodeyatra/reporters/CustomHtmlReporter.java
package com.mycodeyatra.reporters;
 
import org.junit.platform.launcher.TestExecutionListener;
import org.junit.platform.launcher.TestIdentifier;
import org.junit.platform.launcher.TestPlan;
import org.junit.platform.engine.TestExecutionResult;
 
import java.io.FileWriter;
import java.io.IOException;
 
public class CustomHtmlReporter implements TestExecutionListener {
    private final StringBuilder htmlBuilder = new StringBuilder();
    private int passed = 0;
    private int failed = 0;
 
    @Override
    public void testPlanExecutionStarted(TestPlan testPlan) {
        htmlBuilder.append("<!DOCTYPE html><html><head><title>Test Report</title>")
                   .append("<style>body{font-family:sans-serif;} .pass{color:green;} .fail{color:red;}</style>")
                   .append("</head><body><h1>Playwright Custom Execution Summary</h1><table><tr><th>Test</th><th>Status</th></tr>");
    }
 
    @Override
    public void executionFinished(TestIdentifier testIdentifier, TestExecutionResult testExecutionResult) {
        if (testIdentifier.isTest()) {
            String name = testIdentifier.getDisplayName();
            if (testExecutionResult.getStatus() == TestExecutionResult.Status.SUCCESSFUL) {
                passed++;
                htmlBuilder.append("<tr><td>").append(name).append("</td><td class='pass'>PASSED</td></tr>");
            } else {
                failed++;
                htmlBuilder.append("<tr><td>").append(name).append("</td><td class='fail'>FAILED</td></tr>");
            }
        }
    }
 
    @Override
    public void testPlanExecutionFinished(TestPlan testPlan) {
        htmlBuilder.append("</table><h2>Summary: Passed=").append(passed).append(", Failed=").append(failed).append("</h2></body></html>");
        try (FileWriter writer = new FileWriter("target/custom-report.html")) {
            writer.write(htmlBuilder.toString());
        } catch (IOException e) {
            System.err.println("Failed to write custom HTML report: " + e.getMessage());
        }
    }
}
```

---

## 3. Playwright Custom Reporter Execution Test

```java
// src/test/java/com/mycodeyatra/tests/CustomReportExecutionTest.java
package com.mycodeyatra.tests;
 
import com.microsoft.playwright.*;
import org.junit.jupiter.api.*;
 
import static org.junit.jupiter.api.Assertions.*;
 
public class CustomReportExecutionTest {
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
    @DisplayName("Validate Navigation for Custom Reporter Registration")
    void testNavigation() {
        page.navigate("https://mycodeyatra.com");
        assertEquals("MyCodeYatra", page.title());
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

- **Zero Heavy Dependencies**: Custom reporters require no heavy reporting servers or complex plugin installations.
- **CI Artifact Integration**: Publish `target/custom-report.html` as a build artifact in GitHub Actions or Jenkins.
