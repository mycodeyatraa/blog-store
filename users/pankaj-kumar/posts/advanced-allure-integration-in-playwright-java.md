---
id: "post-744"
title: "Advanced Allure Integration in Playwright Java"
slug: "advanced-allure-integration-in-playwright-java"
date: "26-Aug-2026"
author: "pankaj-kumar"
series: "playwright-java-reporting-obs"
seriesOrder: 2
topic: "2. Advanced Allure Features"
tags: ["Playwright", "Java", "Allure", "Advanced", "Trends"]
category: "Automation Testing"
categories: ["Automation Testing", "Playwright Java", "Allure Reporting"]
excerpt: "Master advanced Allure features in Playwright Java: environment properties, link annotations, historical trend tracking, and custom attachments."
readTime: "8 min read"
---

# Advanced Allure Integration in Playwright Java

Beyond basic step logging, advanced Allure reporting incorporates historical trend charts, environment metadata, issue tracking links, and dynamic runtime parameters.

---

## 1. Architectural Overview

Advanced Allure configurations write dynamic `environment.properties` and `executor.json` files to `target/allure-results` prior to report generation.

```
+------------------------------------+
| Playwright Test Environment        |
| Browser: Chromium 124              |
| OS: Windows Server 2025            |
+------------------------------------+
                  |
                  v Writes environment metadata
+------------------------------------+
| target/allure-results              |
| - environment.properties           |
| - executor.json                    |
| - history/ (Trend Copy)            |
+------------------------------------+
```

---

## 2. Environment Properties Writer Utility

```java
// src/main/java/com/mycodeyatra/allure/AllureEnvironmentManager.java
package com.mycodeyatra.allure;
 
import java.io.FileOutputStream;
import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Paths;
import java.util.Properties;
 
public class AllureEnvironmentManager {
 
    public static void setEnvironmentProperties(String browserName, String envUrl, String executionMode) {
        try {
            Files.createDirectories(Paths.get("target/allure-results"));
            Properties props = new Properties();
            props.setProperty("Browser", browserName);
            props.setProperty("Target Environment", envUrl);
            props.setProperty("Execution Mode", executionMode);
            props.setProperty("Java Version", System.getProperty("java.version"));
            props.setProperty("OS", System.getProperty("os.name"));
 
            try (FileOutputStream fos = new FileOutputStream("target/allure-results/environment.properties")) {
                props.store(fos, "Allure Environment Properties");
            }
        } catch (IOException e) {
            System.err.println("Failed to write environment properties: " + e.getMessage());
        }
    }
}
```

---

## 3. Playwright Advanced Allure Test Suite

```java
// src/test/java/com/mycodeyatra/tests/AdvancedAllureTest.java
package com.mycodeyatra.tests;
 
import com.microsoft.playwright.*;
import com.mycodeyatra.allure.AllureEnvironmentManager;
import io.qameta.allure.*;
import org.junit.jupiter.api.*;
 
import static org.junit.jupiter.api.Assertions.*;
 
@Epic("Enterprise Billing")
@Feature("Payment Gateway")
public class AdvancedAllureTest {
    private static Playwright playwright;
    private static Browser browser;
    private Page page;
 
    @BeforeAll
    static void setup() {
        playwright = Playwright.create();
        browser = playwright.chromium().launch(new BrowserType.LaunchOptions().setHeadless(true));
        AllureEnvironmentManager.setEnvironmentProperties("Chromium 124", "https://staging.mycodeyatra.com", "Headless CI");
    }
 
    @BeforeEach
    void init() {
        page = browser.newPage();
    }
 
    @Test
    @Story("Credit Card Transaction")
    @Issue("BUG-4091")
    @TmsLink("TMS-1082")
    @Link(name = "Payment Gateway Docs", url = "https://mycodeyatra.com/docs/payment")
    @Severity(SeverityLevel.BLOCKER)
    void testCreditCardProcessing() {
        page.navigate("https://mycodeyatra.com/pay");
        page.fill("#card-number", "4111111111111111");
        page.click("#pay-now-btn");
 
        Allure.parameter("Payment Type", "Visa");
        Allure.parameter("Amount", "$250.00");
 
        assertTrue(page.isVisible(".success-badge"));
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

- **Issue & TMS Annotations**: Link test failures directly to Jira tickets (`@Issue`) and test management cases (`@TmsLink`).
- **History Preservation**: Copy `target/allure-report/history` into `target/allure-results/history` between CI runs to maintain trend charts.
