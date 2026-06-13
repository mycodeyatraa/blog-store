---
title: Generating Accessibility Reports: JSON and HTML
date: 21-Oct-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [selenium, java, accessibility, axe-core, reporting, ci-cd]
category: UI Automation
categories: [UI Automation, Selenium, Java]
excerpt: >-
  Make accessibility visible to your entire team. Learn how to export Axe-Core results as JSON for CI/CD metrics and compile them into beautiful HTML dashboards using axe-html-reporter.
readTime: 5 min read
---

# Generating Accessibility Reports: JSON and HTML

In our previous article, we successfully integrated Axe-Core into our Selenium framework. When a WCAG violation was detected, we printed the details directly to the console and failed the test via a TestNG assertion.

While this is great for a developer debugging locally, it is completely inadequate for a CI/CD pipeline. Product managers and UI designers don't read Jenkins console logs. They need a persistent, human-readable report that shows exactly what elements failed and how to fix them.

In this article, we will learn how to extract Axe-Core results into JSON and convert them into beautiful HTML Accessibility Reports!

---

## 1. Writing Axe Results to JSON

The `Results` object returned by `AxeBuilder.analyze()` can easily be serialized into JSON. This raw JSON file is an excellent artifact to store in your CI/CD pipeline (like GitHub Actions or Jenkins), as it allows you to track your overall accessibility health metrics over time using tools like Splunk or Datadog.

Deque provides a built-in `AxeReporter` class to handle this serialization:

```java
package com.mycodeyatra.accessibility;
 
import com.deque.html.axecore.selenium.AxeBuilder;
import com.deque.html.axecore.selenium.AxeReporter;
import com.deque.html.axecore.results.Results;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.testng.annotations.AfterMethod;
import org.testng.annotations.BeforeMethod;
import org.testng.annotations.Test;
 
public class JsonReportTest {
 
    private WebDriver driver;
 
    @BeforeMethod
    public void setup() {
        driver = new ChromeDriver();
    }
 
    @Test
    public void generateJsonReport() {
        driver.get("https://mycodeyatra.com/dashboard");
 
        // 1. Run the scan
        Results scanResults = new AxeBuilder().analyze(driver);
 
        // 2. Write the raw JSON output to the target directory
        String reportPath = "target/axe-reports/dashboard_a11y";
        AxeReporter.writeResultsToJsonFile(reportPath, scanResults);
 
        System.out.println("JSON Report generated at: " + reportPath + ".json");
    }
 
    @AfterMethod
    public void teardown() {
        driver.quit();
    }
}
```

---

## 2. Generating a Human-Readable HTML Report

While JSON is great for machines, humans need a dashboard. There are several open-source libraries that parse the Axe JSON output and convert it into a styled HTML page.

One of the most popular is the `axe-html-reporter`. Because it's an NPM package, we can trigger it directly from our Java framework using the `Runtime.getRuntime().exec()` command after our JSON file is generated!

First, ensure the reporter is installed on your CI agent:
```bash
npm install -g axe-html-reporter
```

Next, let's update our Selenium test to automatically generate the HTML dashboard:

```java
import java.io.BufferedReader;
import java.io.InputStreamReader;
 
public class HtmlReportGenerator {
 
    public static void generateHtmlFromAxeJson(String jsonFilePath, String outputHtmlDir) {
        try {
            // Command: axe-html-reporter path/to/results.json --outputDir path/to/output
            String command = String.format("axe-html-reporter %s --outputDir %s", jsonFilePath, outputHtmlDir);
 
            Process process = Runtime.getRuntime().exec(command);
            process.waitFor();
 
            // Print output for debugging
            BufferedReader reader = new BufferedReader(new InputStreamReader(process.getInputStream()));
            String line;
            while ((line = reader.readLine()) != null) {
                System.out.println(line);
            }
 
            System.out.println("HTML Accessibility Report successfully generated in: " + outputHtmlDir);
 
        } catch (Exception e) {
            System.err.println("Failed to generate HTML report: " + e.getMessage());
        }
    }
}
```

Now, instead of staring at a terminal wall of text, your UI designers can open `target/axe-reports/dashboard_a11y.html` and view a beautifully formatted table highlighting the exact DOM elements that failed the contrast ratio checks or are missing ARIA labels!

---

## System Architecture

Here is the data flow for an Enterprise Accessibility Reporting pipeline:

![diagram_1](https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/posts/generating-accessibility-reports-json-html/images/diagram_1.png)

## Conclusion

By standardizing your Axe-Core output into JSON and HTML artifacts, you ensure that accessibility becomes a visible, trackable metric for your entire engineering team, rather than just a pass/fail console log that only the QA team understands.

However, scanning the DOM is only half the battle. In our next article, we will explore **Keyboard Navigation**—writing Selenium scripts that completely abandon the mouse to ensure users with motor disabilities can traverse your application using only the `Tab` and `Enter` keys!
