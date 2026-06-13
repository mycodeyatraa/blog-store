---
title: Axe-Core Integration: Automated WCAG compliance with Selenium
date: 18-Oct-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [selenium, java, accessibility, axe-core, a11y, wcag]
category: UI Automation
categories: [UI Automation, Selenium, Java]
excerpt: >-
  Stop manually checking for ARIA labels and color contrast. Learn how to inject the Axe-Core JavaScript engine into your Selenium framework to instantly catch accessibility violations.
readTime: 5 min read
---

# Axe-Core Integration: Automated WCAG Compliance

In our previous article, we discussed the legal and ethical necessity of the WCAG (Web Content Accessibility Guidelines) standard. We learned that while full compliance requires manual testing, up to 50% of accessibility violations can be caught automatically by inspecting the DOM.

But how exactly do we inspect the DOM for accessibility flaws? Writing custom Selenium `findElements` loops to check every `<img>` for an `alt` attribute would be a nightmare.

Instead, we will use **Axe-Core**, the industry-standard accessibility engine built by Deque Systems.

---

## 1. What is Axe-Core?

Axe-Core is an open-source JavaScript library that executes a massive suite of accessibility rules against a web page. It checks for color contrast, ARIA label validity, semantic HTML structure, and much more. 

Because it is a Javascript library, we can easily inject it into our Selenium WebDriver session, let it scan the page, and return the results back to our Java test!

---

## 2. Setting Up the Axe-Selenium SDK

Deque Systems provides a wrapper library specifically for Selenium Java, which handles the JavaScript injection for us automatically.

Add the following dependency to your `pom.xml`:

```xml
<dependency>
    <groupId>com.deque.html.axe-core</groupId>
    <artifactId>selenium</artifactId>
    <version>4.7.0</version>
</dependency>
```

---

## 3. Writing Your First Accessibility Test

Running an Axe scan is incredibly straightforward. You initialize an `AxeBuilder`, pass it your WebDriver instance, and call `analyze()`.

```java
package com.mycodeyatra.accessibility;
 
import com.deque.html.axecore.selenium.AxeBuilder;
import com.deque.html.axecore.selenium.AxeReporter;
import com.deque.html.axecore.results.Results;
import com.deque.html.axecore.results.Rule;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.testng.Assert;
import org.testng.annotations.AfterMethod;
import org.testng.annotations.BeforeMethod;
import org.testng.annotations.Test;
 
import java.util.List;
 
public class WcagComplianceTest {
 
    private WebDriver driver;
 
    @BeforeMethod
    public void setup() {
        driver = new ChromeDriver();
    }
 
    @Test
    public void verifyDashboardAccessibility() {
        // 1. Navigate to the page you want to test
        driver.get("https://mycodeyatra.com/dashboard");
 
        // 2. Initialize the AxeBuilder and run the scan
        AxeBuilder builder = new AxeBuilder();
        Results scanResults = builder.analyze(driver);
 
        // 3. Extract the list of violations
        List<Rule> violations = scanResults.getViolations();
 
        // 4. Print detailed error messages if violations exist
        if (!violations.isEmpty()) {
            System.out.println("Found " + violations.size() + " Accessibility Violations:");
 
            for (Rule violation : violations) {
                System.out.println("-------------------------------------------------");
                System.out.println("Rule ID: " + violation.getId());
                System.out.println("Impact: " + violation.getImpact());
                System.out.println("Description: " + violation.getDescription());
                System.out.println("Help URL: " + violation.getHelpUrl());
            }
        }
 
        // 5. Fail the test if there is even a single violation
        Assert.assertEquals(violations.size(), 0, "WCAG AA Violations found on Dashboard!");
    }
 
    @AfterMethod
    public void teardown() {
        driver.quit();
    }
}
```

---

## 4. Understanding the Violation Object

If the test fails, the output will guide your developers exactly to the problem. An example Axe-Core output looks like this:

```text
Rule ID: color-contrast
Impact: serious
Description: Ensures the contrast between foreground and background colors meets WCAG 2 AA contrast ratio thresholds
Help URL: https://dequeuniversity.com/rules/axe/4.7/color-contrast
```

Notice that the engine not only tells you *what* is broken, but provides a `Help URL` directly to Deque University, explaining exactly *why* it is an issue and providing code snippets on how the frontend developers can fix it!

---

## System Architecture

Here is the data flow of the Axe-Core injection process:

![diagram_1](https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/posts/axe-core-integration-automated-wcag-compliance/images/diagram_1.png)

## Conclusion

By integrating Axe-Core into your Selenium framework, you transform your automated test suite into a strict gatekeeper for accessibility standards. Every time a developer merges code, the build will automatically fail if they introduced a low-contrast button or forgot an ARIA label.

However, viewing violations in the terminal console is not ideal for large enterprise projects. In our next article, we will learn how to parse the Axe `Results` object to **Generate beautiful HTML Accessibility Reports**!
