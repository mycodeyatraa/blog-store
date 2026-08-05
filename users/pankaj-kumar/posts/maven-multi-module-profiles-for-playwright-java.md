---
id: "post-761"
title: "Maven Multi-Module Profiles for Playwright Java"
slug: "maven-multi-module-profiles-for-playwright-java"
date: "12-Sep-2026"
author: "pankaj-kumar"
series: "playwright-java-cicd-docker-ai"
seriesOrder: 7
topic: "7. Maven Profiles & Multi-Module Architecture"
tags: ["Playwright", "Java", "Maven", "Multi-Module", "Profiles"]
category: "Automation Testing"
categories: ["Automation Testing", "Playwright Java", "Maven Architecture"]
excerpt: "Structure enterprise Playwright Java projects into modular Maven architectures with environment-specific execution profiles."
readTime: "8 min read"
---

# Maven Multi-Module Profiles for Playwright Java

Organizing enterprise test suites into multi-module Maven projects separates core page objects, utilities, and API models from test suites.

---

## 1. Multi-Module Project Structure

```
enterprise-automation/
├── pom.xml (Root POM)
├── automation-core/
│   ├── src/main/java/com/mycodeyatra/core/ (Playwright Drivers & Base Pages)
├── automation-api/
│   ├── src/main/java/com/mycodeyatra/api/ (GraphQL & REST API Clients)
└── automation-tests/
    └── src/test/java/com/mycodeyatra/tests/ (UI & E2E Test Suites)
```

---

## 2. Parent pom.xml Profile Configuration

```xml
<!-- Root pom.xml snippet -->
<project>
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.mycodeyatra</groupId>
    <artifactId>enterprise-automation</artifactId>
    <version>1.0.0-SNAPSHOT</version>
    <packaging>pom</packaging>
 
    <modules>
        <module>automation-core</module>
        <module>automation-api</module>
        <module>automation-tests</module>
    </modules>
 
    <profiles>
        <profile>
            <id>staging</id>
            <properties>
                <env.url>https://staging.mycodeyatra.com</env.url>
            </properties>
        </profile>
        <profile>
            <id>production</id>
            <properties>
                <env.url>https://mycodeyatra.com</env.url>
            </properties>
        </profile>
    </profiles>
</project>
```

---

## 3. Playwright Multi-Module Test Suite

```java
// automation-tests/src/test/java/com/mycodeyatra/tests/MultiModuleTest.java
package com.mycodeyatra.tests;
 
import com.microsoft.playwright.*;
import org.junit.jupiter.api.*;
 
import static org.junit.jupiter.api.Assertions.*;
 
public class MultiModuleTest {
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
    @DisplayName("Verify Execution with Maven Profile URL")
    void testProfileUrlNavigation() {
        String targetUrl = System.getProperty("env.url", "https://mycodeyatra.com");
        page.navigate(targetUrl);
        assertTrue(page.url().contains("mycodeyatra"));
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

- **Module Isolation**: `automation-core` should have zero dependencies on test frameworks like JUnit, keeping core utilities framework-agnostic.
- **Profile Activation**: Trigger environment profiles via `mvn test -Pstaging` or `mvn test -Pproduction`.
