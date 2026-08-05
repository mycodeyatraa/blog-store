---
title: "Generating Living Documentation and HTML Reports with Cucumber Java"
date: "18-Aug-2026"
lastUpdated: "18-Aug-2026"
author: "pankaj-kumar"
authorName: "Pankaj Kumar"
authorRole: "Automation Architect"
authorAvatar: "https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG"
authorBio: "Automation Architect"
authorGithub: "https://github.com/pankajhyd"
authorLinkedin: "https://www.linkedin.com/in/pankaj-kumar-94a2b227/"
tags: ["Playwright", "Java", "Cucumber", "Reporting", "Allure"]
category: "Automation Testing"
categories: ["Automation Testing", "Playwright Java", "Living Documentation"]
excerpt: "Generate interactive HTML reports, Allure BDD documentation, and living test artifacts from Playwright Java Cucumber executions."
readTime: "7 min read"
---

# Generating Living Documentation and HTML Reports with Cucumber Java

Living documentation ensures system requirements and automated test results remain continuously in sync, serving as a single source of truth for both engineering and product stakeholders.

---

## 1. Native Cucumber HTML Plugin Setup

```java
// src/test/java/com/mycodeyatra/runner/ReportRunner.java
package com.mycodeyatra.runner;
 
import org.junit.platform.suite.api.*;
import static io.cucumber.junit.platform.engine.Constants.*;
 
@Suite
@IncludeEngines("cucumber")
@SelectClasspathResource("features")
@ConfigurationParameter(key = GLUE_PROPERTY_NAME, value = "com.mycodeyatra.steps")
@ConfigurationParameter(key = PLUGIN_PROPERTY_NAME, 
    value = "pretty, html:target/cucumber-reports/cucumber.html, json:target/cucumber-reports/cucumber.json")
public class ReportRunner {
}
```

---

## 2. Integrating Allure Cucumber-JVM Reporter

Add Allure dependency to `pom.xml`:

```xml
<dependency>
    <groupId>io.qameta.allure</groupId>
    <artifactId>allure-cucumber7-jvm</artifactId>
    <version>2.27.0</version>
    <scope>test</scope>
</dependency>
```

Add plugin parameter:

```java
@ConfigurationParameter(key = PLUGIN_PROPERTY_NAME, value = "io.qameta.allure.cucumber7jvm.AllureCucumber7Jvm")
```

---

## 3. Attaching Playwright Screenshots and Videos to Allure

```java
// Inside Hooks.java
@After
public void attachArtifacts(Scenario scenario) {
    if (scenario.isFailed()) {
        byte[] screenshot = page.screenshot(new Page.ScreenshotOptions().setFullPage(true));
        Allure.addAttachment("Failure Screenshot", new ByteArrayInputStream(screenshot));
    }
}
```

Generate and view Allure living documentation:

```bash
# Generate report files
mvn clean test
 
# Open interactive Allure Living Documentation in browser
allure serve target/allure-results
```

---

## 4. Benefits of Living Documentation

- **Self-Updating**: Requirements documentation updates automatically upon every test suite execution in CI/CD.
- **Traceability**: Easily trace failed Gherkin steps directly back to corresponding Playwright browser action logs and video recordings.
