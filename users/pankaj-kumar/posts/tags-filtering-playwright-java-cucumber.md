---
title: "Categorizing and Filtering Tests with Cucumber Tags in Java"
date: "16-Aug-2026"
lastUpdated: "16-Aug-2026"
author: "pankaj-kumar"
authorName: "Pankaj Kumar"
authorRole: "Automation Architect"
authorAvatar: "https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG"
authorBio: "Automation Architect"
authorGithub: "https://github.com/pankajhyd"
authorLinkedin: "https://www.linkedin.com/in/pankaj-kumar-94a2b227/"
tags: ["Playwright", "Java", "Cucumber", "Tags", "CLI"]
category: "Automation Testing"
categories: ["Automation Testing", "Playwright Java", "Tags"]
excerpt: "Filter, group, and execute targeted test subsets (Smoke, Regression, Sanity, Flaky) using Cucumber tags and Maven command-line parameters in Playwright Java."
readTime: "7 min read"
---

# Categorizing and Filtering Tests with Cucumber Tags in Java

Tags allow you to organize scenarios into logical suites and selectively trigger runs based on CI/CD pipeline triggers (e.g. running `@smoke` tests on pull requests vs `@regression` overnight).

---

## 1. Applying Tags to Scenarios and Features

```gherkin
# src/test/resources/features/checkout.feature
@Checkout @Severity-High
Feature: Checkout Process
 
  @Smoke @P0
  Scenario: Rapid guest checkout with credit card
    Given guest user has added item to cart
    When completes payment with valid card
    Then order confirmation should be displayed
 
  @Regression @P1 @WIP
  Scenario: Applying promo code during checkout
    Given registered user has items in cart
    When applies promo code "SAVE20"
    Then cart total should reflect 20 percent discount
```

---

## 2. Command Line Tag Expressions

Execute specific suites via Maven terminal arguments:

```bash
# Run only @Smoke tests
mvn test -Dcucumber.filter.tags="@Smoke"
 
# Run tests tagged @Smoke AND NOT @WIP
mvn test -Dcucumber.filter.tags="@Smoke and not @WIP"
 
# Run tests tagged @Smoke OR @Sanity
mvn test -Dcucumber.filter.tags="@Smoke or @Sanity"
 
# Complex tag logic
mvn test -Dcucumber.filter.tags="(@Regression or @Checkout) and not @Flaky"
```

---

## 3. Programmatic Tag Configuration in JUnit 5 Runner

```java
// src/test/java/com/mycodeyatra/runner/SmokeRunner.java
package com.mycodeyatra.runner;
 
import org.junit.platform.suite.api.*;
 
import static io.cucumber.junit.platform.engine.Constants.*;
 
@Suite
@IncludeEngines("cucumber")
@SelectClasspathResource("features")
@ConfigurationParameter(key = GLUE_PROPERTY_NAME, value = "com.mycodeyatra.steps")
@ConfigurationParameter(key = FILTER_TAGS_PROPERTY_NAME, value = "@Smoke and not @Flaky")
public class SmokeRunner {
}
```

---

## 4. Tag Naming Conventions Table

| Tag Category | Example Tags | Purpose |
| :--- | :--- | :--- |
| **Suite Level** | `@Smoke`, `@Regression`, `@Sanity` | Pipeline triggers |
| **Priority** | `@P0`, `@P1`, `@P2` | Severity classification |
| **Status** | `@WIP`, `@Flaky`, `@Quarantine` | Test stability status |
| **Module** | `@Auth`, `@Billing`, `@Settings` | Domain mapping |
