---
id: "post-743"
title: "Allure Basics in Playwright Java Frameworks"
slug: "allure-basics-in-playwright-java-frameworks"
date: "25-Aug-2026"
author: "pankaj-kumar"
series: "playwright-java-reporting-obs"
seriesOrder: 1
topic: "1. Allure Reporting Basics"
tags: ["Playwright", "Java", "Allure", "JUnit 5", "Reporting"]
category: "Automation Testing"
categories: ["Automation Testing", "Playwright Java", "Allure Reporting"]
excerpt: "Integrate Allure Reporting into Playwright Java with JUnit 5. Capture test steps, screenshots, and execution metadata."
readTime: "8 min read"
---

# Allure Basics in Playwright Java Frameworks

Visual reporting is critical for automation suites. Allure Framework provides rich HTML reporting with execution timelines, attachment handling, and step breakdowns.

---

## 1. Architectural Overview

The Playwright Java framework uses Allure JUnit 5 listeners to record test events, screenshots, trace zips, and step annotations during execution.

```
+------------------------------------+       +------------------------------------+
| Playwright Java (JUnit 5 Runner)   | ----> | Allure JUnit 5 Extension           |
| (@Test, @Step, Playwright API)     |       | (Captures Results & Attachments)   |
+------------------------------------+       +------------------------------------+
                                                        |
                                                        v Writes JSON/XML to target/allure-results
+---------------------------------------------------------------------------------+
| Allure Commandline Engine (Generates Interactive Single-Page HTML Report)       |
+---------------------------------------------------------------------------------+
```

---

## 2. Allure Configuration & Test Listener

```java
// src/test/java/com/mycodeyatra/listeners/AllureTestListener.java
package com.mycodeyatra.listeners;
 
import io.qameta.allure.Allure;
import com.microsoft.playwright.Page;
import org.junit.jupiter.api.extension.ExtensionContext;
import org.junit.jupiter.api.extension.TestWatcher;
 
import java.io.ByteArrayInputStream;
 
public class AllureTestListener implements TestWatcher {
 
    public static void attachScreenshot(Page page, String name) {
        byte[] screenshot = page.screenshot(new Page.ScreenshotOptions().setFullPage(true));
        Allure.addAttachment(name, "image/png", new ByteArrayInputStream(screenshot), ".png");
    }
 
    @Override
    public void testFailed(ExtensionContext context, Throwable cause) {
        Allure.addAttachment("Failure Reason", cause.getMessage());
    }
}
```

---

## 3. Playwright Allure Basic Test Suite

```java
// src/test/java/com/mycodeyatra/tests/AllureBasicsTest.java
package com.mycodeyatra.tests;
 
import com.microsoft.playwright.*;
import com.mycodeyatra.listeners.AllureTestListener;
import io.qameta.allure.*;
import org.junit.jupiter.api.*;
import org.junit.jupiter.api.extension.ExtendWith;
 
import static org.junit.jupiter.api.Assertions.*;
 
@Epic("E-Commerce Portal")
@Feature("User Authentication")
@ExtendWith(AllureTestListener.class)
public class AllureBasicsTest {
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
    @Story("Valid Login Flow")
    @Severity(SeverityLevel.CRITICAL)
    @Description("Verify that valid credentials navigate user to dashboard with screenshot attachment.")
    void testValidLogin() {
        stepNavigateToLogin();
        stepPerformLogin("qa_admin", "AdminPass123!");
        stepVerifyDashboard();
    }
 
    @Step("Navigate to Login Page")
    private void stepNavigateToLogin() {
        page.navigate("https://mycodeyatra.com/login");
    }
 
    @Step("Perform Login for User: {username}")
    private void stepPerformLogin(String username, String password) {
        page.fill("#username", username);
        page.fill("#password", password);
        page.click("#login-btn");
    }
 
    @Step("Verify Dashboard Page Loaded")
    private void stepVerifyDashboard() {
        assertTrue(page.isVisible("#dashboard-container"));
        AllureTestListener.attachScreenshot(page, "Dashboard Successful State");
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

| Metric | Standard JUnit Console Output | Allure HTML Reporting |
| :--- | :--- | :--- |
| **Visual Evidence** | Text logs only | Full-page PNG & Playwright Trace zip files |
| **Stakeholder Utility** | Developer focused | Executive & QA manager friendly |
| **Categorization** | Package names | Epic, Feature, Story, and Severity tagging |
