---
title: "Data-Driven Testing using Scenario Outlines in Playwright Java"
date: "17-Aug-2026"
lastUpdated: "17-Aug-2026"
author: "pankaj-kumar"
authorName: "Pankaj Kumar"
authorRole: "Automation Architect"
authorAvatar: "https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG"
authorBio: "Automation Architect"
authorGithub: "https://github.com/pankajhyd"
authorLinkedin: "https://www.linkedin.com/in/pankaj-kumar-94a2b227/"
tags: ["Playwright", "Java", "Cucumber", "Scenario Outlines", "Data Driven"]
category: "Automation Testing"
categories: ["Automation Testing", "Playwright Java", "Scenario Outlines"]
excerpt: "Implement data-driven BDD tests using Scenario Outlines and Examples tables in Cucumber-JVM with Playwright Java."
readTime: "7 min read"
---

# Data-Driven Testing using Scenario Outlines in Playwright Java

Scenario Outlines allow you to execute the same test logic multiple times using different datasets defined in an `Examples:` table.

---

## 1. Scenario Outline Syntax

```gherkin
# src/test/resources/features/login_datadriven.feature
Feature: Data-Driven Login Validations
 
  Scenario Outline: Validate login behavior with multiple user accounts
    Given the user opens the application login page
    When user submits credentials with username "<username>" and password "<password>"
    Then the login result should display status "<status>" and message "<message>"
 
    Examples: Valid Credentials
      | username    | password    | status  | message              |
      | admin_user  | secret123   | SUCCESS | Welcome Admin        |
      | standard_qa | qa_pass_2026| SUCCESS | Welcome QA Tester    |
 
    Examples: Invalid Credentials
      | username    | password    | status  | message              |
      | invalid_usr | secret123   | FAILURE | Invalid Credentials  |
      | admin_user  | wrong_pass  | FAILURE | Invalid Credentials  |
      |             | secret123   | FAILURE | Username Required    |
```

---

## 2. Playwright Java Step Definitions for Outlines

```java
// src/test/java/com/mycodeyatra/steps/LoginOutlineSteps.java
package com.mycodeyatra.steps;
 
import com.microsoft.playwright.Page;
import io.cucumber.java.en.*;
import static org.junit.jupiter.api.Assertions.*;
 
public class LoginOutlineSteps {
    private Page page;
 
    public LoginOutlineSteps(Hooks hooks) {
        this.page = hooks.getPage();
    }
 
    @Given("the user opens the application login page")
    public void openLoginPage() {
        page.navigate("https://mycodeyatra.com/login");
    }
 
    @When("user submits credentials with username {string} and password {string}")
    public void enterCredentials(String username, String password) {
        page.fill("#username", username);
        page.fill("#password", password);
        page.click("#submit-button");
    }
 
    @Then("the login result should display status {string} and message {string}")
    public void verifyResult(String expectedStatus, String expectedMessage) {
        if ("SUCCESS".equalsIgnoreCase(expectedStatus)) {
            assertTrue(page.isVisible("#dashboard-container"));
            assertEquals(expectedMessage, page.textContent("#welcome-banner"));
        } else {
            assertTrue(page.isVisible(".error-alert"));
            assertEquals(expectedMessage, page.textContent(".error-alert"));
        }
    }
}
```

---

## 3. Advantages of Multiple Examples Tables

- **Logical Separation**: Separate positive test vectors from boundary and negative test vectors without repeating scenario steps.
- **Reporting Granularity**: Test reports list every row as a distinct scenario iteration (e.g. `Scenario Outline: Validate login behavior... #1`, `#2`, `#3`).
