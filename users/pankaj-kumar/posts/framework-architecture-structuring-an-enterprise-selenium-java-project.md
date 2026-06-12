---
title: Framework Architecture: Structuring an Enterprise Selenium Java Project
date: 02-Aug-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [selenium, java, architecture, framework, test-automation]
category: UI Automation
categories: [UI Automation, Selenium, Java]
excerpt: >-
  Design an enterprise-grade folder structure for a Selenium Java framework. Leverage the standard Maven directory layout and enforce strict Separation of Concerns (SoC) for scalable test automation.
readTime: 6 min read
---

# Framework Architecture: Structuring an Enterprise Selenium Java Project

A test framework that starts as a few scripts in a single package will quickly degrade into an unmaintainable monolith. As your test suite scales to hundreds or thousands of tests, the architecture of your project becomes just as important as the code itself. 

In this article, we'll design an enterprise-grade folder structure for a Selenium Java framework. We'll leverage the standard Maven directory layout and enforce strict Separation of Concerns (SoC).

---

## The Philosophy of Enterprise Architecture

An enterprise test framework must decouple test logic from test execution, test data, and configuration. 

> **Key insight:** A test class should only contain test steps and assertions. It should know absolutely nothing about how a WebDriver is initialized, where properties are loaded from, or how a button is located on the page.

To achieve this, we divide our framework into logical layers.

---

## The Maven Standard Directory Layout

Maven dictates a standard directory layout that universally separates application/framework code from test code. We will build upon this foundation.

![diagram_1](https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/posts/framework-architecture-structuring-an-enterprise-selenium-java-project/images/diagram_1.png)

### 1. The Core Engine: `src/main/java`

This directory houses the engine of your framework. Nothing in this directory should contain a `@Test` annotation.

#### `com.mycodeyatra.core`
This package contains the foundational building blocks of the framework.
- `DriverFactory.java`: Responsible for initializing and returning ThreadLocal WebDriver instances.
- `ConfigLoader.java`: Reads properties from environment files.
- `BasePage.java`: A wrapper around standard Selenium actions (click, type, wait) with built-in explicit waits and logging.

#### `com.mycodeyatra.pages`
This is where your Page Object Model (POM) classes live. 
- `LoginPage.java`
- `DashboardPage.java`
- Each class contains locators (`By` objects or `@FindBy`) and actions specific to that page.

#### `com.mycodeyatra.utils`
Reusable utility classes that don't depend on WebDriver.
- `ExcelReader.java`: Parses data-driven files.
- `StringModifiers.java`: Generates random emails or formats strings.
- `DateUtils.java`: Handles timestamp generation for logging and reporting.

---

### 2. The Test Suite: `src/test/java`

This directory is exclusively for test execution and reporting listeners.

#### `com.mycodeyatra.tests`
Your actual test classes. They should be clean, readable, and focused solely on business logic.
- `BaseTest.java`: Handles `@BeforeMethod` (driver setup) and `@AfterMethod` (driver teardown).
- `LoginTest.java`: Extends `BaseTest` and executes login scenarios.

#### `com.mycodeyatra.listeners`
TestNG listeners that monitor the test execution lifecycle.
- `TestListener.java`: Implements `ITestListener`. Captures screenshots on failure and attaches them to the report.
- `AnnotationTransformer.java`: Dynamically attaches retry analyzers to all tests.

---

### 3. Configuration & Data: `src/test/resources`

Code belongs in `.java` files; configuration and data belong in resources. 

![diagram_2](https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/posts/framework-architecture-structuring-an-enterprise-selenium-java-project/images/diagram_2.png)

- **`config.properties`**: Global configuration (e.g., default timeout, implicit wait, browser type).
- **Environment Files (`qa.properties`, `staging.properties`)**: Environment-specific URLs and credentials.
- **`testng.xml`**: Test execution runners. You might have `smoke.xml`, `regression.xml`, etc.
- **`testdata/`**: JSON, CSV, or Excel files containing data used by your DataProviders.

---

## Code Example: The Flow of Control

To illustrate why this structure is powerful, look at how clean a test class becomes when responsibilities are properly delegated:

```java
package com.mycodeyatra.tests;
 
import com.mycodeyatra.pages.LoginPage;
import org.testng.Assert;
import org.testng.annotations.Test;
 
public class LoginTest extends BaseTest {
 
    @Test(description = "Verify successful login with valid credentials")
    public void validLoginTest() {
        // Page object is instantiated; BaseTest already set up the driver
        LoginPage loginPage = new LoginPage(driver);
 
        // Test steps are highly readable
        boolean isDashboardDisplayed = loginPage
            .enterUsername("admin")
            .enterPassword("password123")
            .clickLogin()
            .isDashboardHeaderVisible();
 
        // Assertion is decoupled from page logic
        Assert.assertTrue(isDashboardDisplayed, "Dashboard was not displayed after login.");
    }
}
```

Notice what is **missing** from this test:
- No `new ChromeDriver()` (Handled by `BaseTest` / `DriverFactory`)
- No `driver.findElement()` (Handled by `LoginPage`)
- No hardcoded URLs (Handled by `ConfigLoader` in `BaseTest`)

## Key Takeaways

1. **Keep `src/main` and `src/test` strictly separated.** Framework logic goes in `main`; test execution goes in `test`.
2. **Abstract WebDriver initialization.** Tests should ask for a driver, not build one.
3. **Isolate configuration.** Hardcoding URLs or browser names makes CI/CD execution nearly impossible. Use properties files.
4. **Thin tests, thick pages.** Your Page Objects should do the heavy lifting. Your tests should simply orchestrate actions and assert outcomes.

In the next article, we will explore **Spring DI / Testcontainers** to take our enterprise framework to the next level of modularity and modern infrastructure management.
