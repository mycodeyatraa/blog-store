---
title: Making the Invisible Visible: Generating Accessibility Reports with Axe and ExtentReports
date: 17-Jul-2025
lastUpdated: 17-Jul-2025
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [java, selenium, accessibility, a11y, extent-reports, axe-core, test-automation, reporting]
category: Selenium Java
categories: [Selenium Java, Accessibility Testing]
excerpt: >-
  Translate raw Axe-Core JSON data into beautiful, executive-friendly HTML artifacts using ExtentReports. Build custom test loggers to categorize critical WCAG violations natively in your test runs.
readTime: 6 min read
---

# Making the Invisible Visible: Generating Accessibility Reports with Axe and ExtentReports

In our previous tutorials, we integrated the Axe-Core engine into Selenium and printed WCAG violations directly to the standard console output. 

While console logs are great for developers debugging locally, they are completely useless for Product Managers, QA Leads, and Compliance Officers. To prove that your application is legally compliant (e.g., ADA, Section 508), you must generate professional, auditable artifacts.

In this tutorial, we will learn how to extract the raw JSON data from Axe and format it into beautiful, shareable HTML reports using **ExtentReports**!

---

## 1. Setting up ExtentReports

ExtentReports is a massively popular open-source reporting library for Java testing frameworks. It allows us to build gorgeous HTML dashboards with pie charts, logs, and screenshots.

Add the ExtentReports dependency to your `pom.xml`:

```xml
<dependency>
    <groupId>com.aventstack</groupId>
    <artifactId>extentreports</artifactId>
    <version>5.0.9</version>
</dependency>
```

*(Note: We will also use the `Gson` library, which you should already have from the previous Axe tutorials).*

---

## 2. Architecting the Reporting Framework

We are going to use TestNG's `@BeforeSuite` and `@AfterSuite` annotations to initialize and flush the Extent HTML report. 

Inside our test method, we will run the Axe scan, parse the violations, and write a custom log entry to the Extent report for every single WCAG violation found!

```java
import com.aventstack.extentreports.ExtentReports;
import com.aventstack.extentreports.ExtentTest;
import com.aventstack.extentreports.Status;
import com.aventstack.extentreports.reporter.ExtentSparkReporter;
import com.deque.html.axecore.results.Results;
import com.deque.html.axecore.results.Rule;
import com.deque.html.axecore.selenium.AxeBuilder;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.testng.Assert;
import org.testng.annotations.*;
import java.util.List;
public class AxeReportingTest {
    static ExtentReports extent;
    WebDriver driver;
    ExtentTest testLogger;
    @BeforeSuite
    public void setupReport() {
        // 1. Initialize the HTML Reporter
        ExtentSparkReporter spark = new ExtentSparkReporter("target/AccessibilityReport.html");
        extent = new ExtentReports();
        extent.attachReporter(spark);
        // Add some metadata for the executives
        extent.setSystemInfo("Environment", "QA");
        extent.setSystemInfo("WCAG Standard", "2.1 AA");
    }
    @BeforeMethod
    public void setup() {
        driver = new ChromeDriver();
    }
    @Test
    public void testHomepageAccessibility() {
        // 2. Create a test entry in the HTML report
        testLogger = extent.createTest("Homepage WCAG Scan");
        driver.get("https://practice.mycodeyatra.com/");
        testLogger.log(Status.INFO, "Navigated to Homepage");
        // 3. Execute the Axe Scan
        AxeBuilder builder = new AxeBuilder();
        Results axeResults = builder.analyze(driver);
        List<Rule> violations = axeResults.getViolations();
        // 4. Log the results into the Extent Report!
        if (violations.isEmpty()) {
            testLogger.log(Status.PASS, "✅ Zero Accessibility Violations Found!");
        } else {
            testLogger.log(Status.FAIL, "❌ Found " + violations.size() + " Accessibility Violations");
            // Iterate through every violation and format it nicely using HTML tables
            for (Rule v : violations) {
                String htmlLog = "<b>Rule ID:</b> " + v.getId() + "<br>" +
                                 "<b>Impact:</b> " + v.getImpact() + "<br>" +
                                 "<b>Description:</b> " + v.getDescription() + "<br>" +
                                 "<a href='" + v.getHelpUrl() + "' target='_blank'>How to fix this</a>";
                // If it's a critical bug, log it as a FATAL error
                if(v.getImpact().equals("critical")) {
                    testLogger.log(Status.FAIL, "<span style='color:red'>" + htmlLog + "</span>");
                } else {
                    testLogger.log(Status.WARNING, htmlLog);
                }
            }
        }
        // 5. Hard fail the TestNG test so the CI/CD pipeline turns Red
        Assert.assertEquals(violations.size(), 0, "Accessibility bugs detected!");
    }
    @AfterMethod
    public void teardown() {
        driver.quit();
    }
    @AfterSuite
    public void flushReport() {
        // 6. Write the HTML file to the disk!
        extent.flush();
    }
}
```

---

## 3. Saving the Raw JSON Artifact

While ExtentReports generates a beautiful HTML dashboard for humans, you might also want to save the raw JSON data generated by Axe. 

Saving the JSON is extremely useful if you want to pipe your accessibility data into an enterprise dashboard like Datadog, ELK (Elasticsearch), or Grafana!

You can easily serialize the Axe `Results` object directly to a file using Google's `Gson` library:

```java
import com.google.gson.Gson;
import com.google.gson.GsonBuilder;
import java.io.FileWriter;
import java.io.IOException;
public void saveRawJsonArtifact(Results axeResults) {
    // Initialize Gson with pretty printing so the JSON is readable
    Gson gson = new GsonBuilder().setPrettyPrinting().create();
    String jsonOutput = gson.toJson(axeResults);
    // Write the JSON string to a file
    try (FileWriter file = new FileWriter("target/axe-raw-report.json")) {
        file.write(jsonOutput);
        System.out.println("Raw JSON saved to target/axe-raw-report.json");
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```

## Conclusion

By merging the analytical power of **Axe-Core** with the visualization capabilities of **ExtentReports**, you transform invisible DOM violations into actionable, executive-friendly compliance artifacts. 

Whether you are failing the CI/CD pipeline over a critical ARIA bug or exporting raw JSON metrics into a Grafana dashboard, your team now has the visibility required to enforce a culture of accessibility!

In our next tutorial, we will step outside of Selenium and explore how to execute these exact same Axe scans natively from your terminal using **Accessibility CLI Tooling**!
