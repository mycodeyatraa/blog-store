---
title: Writing Your First a11y Test: Integrating Axe-Core with Selenium Java
date: 09-Jul-2026
lastUpdated: 09-Jul-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [java, selenium, accessibility, a11y, axe-core, wcag, test-automation]
category: Selenium Java
categories: [Selenium Java, Accessibility Testing]
excerpt: >-
  Build your first automated accessibility test. Learn how to install the axe-selenium-java SDK, execute WCAG scans across your DOM, and parse the violation JSON reports to catch critical UI bugs.
readTime: 6 min read
---

# Writing Your First a11y Test: Integrating Axe-Core with Selenium Java

In our previous tutorial, we explored the architecture of automated accessibility testing. We learned that because Selenium only validates the physical existence of DOM elements, we must inject a JavaScript engine to analyze the DOM for WCAG (Web Content Accessibility Guidelines) violations.

The absolute best open-source engine for this is **axe-core** by Deque Systems.

In this tutorial, we will write our very first automated Accessibility test by integrating the official Axe-Selenium-Java SDK into our framework!

---

## 1. Setting Up the Project

To use Axe with Selenium Java, we need to add the `axe-selenium` dependency to our `pom.xml`. This library acts as a wrapper around Selenium's `JavascriptExecutor`, automatically fetching the latest `axe.min.js` script and injecting it into the browser.

```xml
<dependency>
    <groupId>com.deque.html.axe-core</groupId>
    <artifactId>selenium</artifactId>
    <version>4.3.1</version>
</dependency>
```

*(Note: We will also use the `Gson` library to parse the JSON results returned by Axe)*

```xml
<dependency>
    <groupId>com.google.code.gson</groupId>
    <artifactId>gson</artifactId>
    <version>2.10.1</version>
</dependency>
```

---

## 2. Writing the Axe Test

The beauty of the Axe Selenium integration is that you can run an accessibility scan at any point during your normal functional tests. 

Let's write a test that navigates to a login page, and then triggers an Axe scan to verify the form is accessible.

```java
import com.deque.html.axecore.results.Results;
import com.deque.html.axecore.results.Rule;
import com.deque.html.axecore.selenium.AxeBuilder;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.testng.Assert;
import org.testng.annotations.AfterMethod;
import org.testng.annotations.BeforeMethod;
import org.testng.annotations.Test;
import java.util.List;
public class AxeAccessibilityTest {
    WebDriver driver;
    @BeforeMethod
    public void setup() {
        driver = new ChromeDriver();
    }
    @Test
    public void testLoginPageAccessibility() {
        // 1. Navigate to the page using standard Selenium
        driver.get("https://practice.mycodeyatra.com/login");
        // 2. Initialize the AxeBuilder and run the scan!
        // Axe will automatically inject the JS engine and execute it
        AxeBuilder builder = new AxeBuilder();
        Results axeResults = builder.analyze(driver);
        // 3. Extract the Violations
        List<Rule> violations = axeResults.getViolations();
        // 4. Print the violations to the console for debugging
        if (violations.size() > 0) {
            System.out.println("❌ Accessibility Violations Found: " + violations.size());
            for (Rule violation : violations) {
                System.out.println("----------------------------------------");
                System.out.println("Rule ID: " + violation.getId());
                System.out.println("Description: " + violation.getDescription());
                System.out.println("Impact: " + violation.getImpact());
                System.out.println("Help URL: " + violation.getHelpUrl());
            }
        }
        // 5. Assert that zero violations exist!
        Assert.assertEquals(violations.size(), 0, 
            "Expected 0 accessibility violations, but found " + violations.size());
    }
    @AfterMethod
    public void teardown() {
        driver.quit();
    }
}
```

---

## 3. Understanding the Axe Output

If the page contains accessibility bugs, your console output will look something like this:

```text
❌ Accessibility Violations Found: 2
----------------------------------------
Rule ID: color-contrast
Description: Ensures the contrast between foreground and background colors meets WCAG 2 AA contrast ratio thresholds
Impact: serious
Help URL: https://dequeuniversity.com/rules/axe/4.3/color-contrast
----------------------------------------
Rule ID: label
Description: Ensures every form element has a label
Impact: critical
Help URL: https://dequeuniversity.com/rules/axe/4.3/label
```

This output is incredibly actionable. 
- The **Rule ID** tells you exactly what WCAG rule was violated.
- The **Description** explains the problem in plain English.
- The **Impact** categorizes the severity (minor, moderate, serious, critical).
- The **Help URL** provides a direct link to Deque University, explaining exactly how the developer can fix the HTML to make it compliant!

---

## 4. Scanning Specific Elements

Running an Axe scan against the entire page `builder.analyze(driver)` is great for initial audits. However, if your application has a persistent navigation bar that is full of accessibility bugs, the scan will fail every single time, masking any new bugs introduced into the main content area.

To prevent this, you can configure Axe to scan only a specific `WebElement`, or explicitly exclude a known broken element.

**Scanning a specific element (e.g., only the login form):**

```java
import org.openqa.selenium.By;
import org.openqa.selenium.WebElement;
WebElement loginForm = driver.findElement(By.id("login-form"));
Results axeResults = builder.analyze(driver, loginForm);
```

**Excluding a known broken element (e.g., skipping the broken footer):**

```java
// Pass the CSS Selector of the element to exclude
AxeBuilder builder = new AxeBuilder().exclude("#legacy-footer");
Results axeResults = builder.analyze(driver);
```

## Conclusion

By integrating Axe-Core into your Selenium tests, you can instantly upgrade your test suite to catch WCAG violations before they reach production. 

However, running a scan is only the first step. Understanding *why* a rule failed is critical to fixing the application. In our next tutorial, we will dive deep into the most common accessibility failure: **ARIA Validation**. We will learn what ARIA tags are, why developers abuse them, and how to test them correctly!
