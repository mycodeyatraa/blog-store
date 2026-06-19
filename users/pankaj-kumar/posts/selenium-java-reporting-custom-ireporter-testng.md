---
title: Under the Hood: Building Custom TestNG Reporters from Scratch
date: 02-Sep-2026
lastUpdated: 02-Sep-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [java, selenium, reporting, ireporter, testng, custom-report, test-automation, architecture]
category: Selenium Java
categories: [Selenium Java, Reporting & Analytics]
excerpt: >-
  When third-party tools are banned, you have to build it yourself. Learn how to implement the TestNG IReporter interface to intercept post-execution data and generate custom JSON, XML, or HTML reports from scratch.
readTime: 6 min read
---

# Under the Hood: Building Custom TestNG Reporters from Scratch

We have spent this entire module relying on third-party libraries. We used Allure to generate interactive dashboards. We used Extent Reports to generate single-page HTML files. We used ReportPortal for AI-powered analytics.

But what if your company’s security policy explicitly forbids third-party reporting libraries? What if your CTO demands a highly specific, proprietary XML format that integrates into an obscure internal dashboard? 

You can't use Allure. You can't use Extent. You have to build it yourself.

In this final capstone of the Reporting and Analytics module, we will pull back the curtain and learn how to implement the TestNG `IReporter` interface to build a **Custom Reporter** from absolute scratch!

---

## 1. ITestListener vs. IReporter

Before we write code, we must understand the architectural difference between the two primary TestNG reporting interfaces:

*   **`ITestListener`:** Executes in *real-time*. The `onTestFailure` method fires the exact millisecond a test fails. We used this earlier to capture Selenium screenshots mid-execution.
*   **`IReporter`:** Executes *post-execution*. The `generateReport` method only fires after the entire 5,000-test suite has completely finished. It is given a massive Java object containing the results of every single test that just ran.

If you want to generate a final summary file (HTML, XML, JSON, CSV), you must use `IReporter`.

---

## 2. Implementing the IReporter Interface

Let's build a Custom Reporter that extracts the execution data and generates a simple proprietary JSON file that our imaginary internal company dashboard requires.

Create a new class named `CustomJsonReporter`:

```java
import org.testng.IReporter;
import org.testng.ISuite;
import org.testng.ISuiteResult;
import org.testng.ITestContext;
import org.testng.xml.XmlSuite;
import java.io.FileWriter;
import java.io.IOException;
import java.util.List;
import java.util.Map;
public class CustomJsonReporter implements IReporter {
    @Override
    public void generateReport(List<XmlSuite> xmlSuites, List<ISuite> suites, String outputDirectory) {
        System.out.println("Starting Custom JSON Report Generation...");
        int totalPassed = 0;
        int totalFailed = 0;
        int totalSkipped = 0;
        long totalDuration = 0;
        // 1. Iterate through all the Test Suites that were executed
        for (ISuite suite : suites) {
            Map<String, ISuiteResult> suiteResults = suite.getResults();
            // 2. Iterate through all the <test> tags inside the suite
            for (ISuiteResult sr : suiteResults.values()) {
                ITestContext tc = sr.getTestContext();
                // Extract metrics
                totalPassed += tc.getPassedTests().getAllResults().size();
                totalFailed += tc.getFailedTests().getAllResults().size();
                totalSkipped += tc.getSkippedTests().getAllResults().size();
                long testStart = tc.getStartDate().getTime();
                long testEnd = tc.getEndDate().getTime();
                totalDuration += (testEnd - testStart);
            }
        }
        // 3. Construct our Custom JSON Payload
        String jsonPayload = String.format(
            "{\n" +
            "  \"reportName\": \"Nightly Regression\",\n" +
            "  \"totalPassed\": %d,\n" +
            "  \"totalFailed\": %d,\n" +
            "  \"totalSkipped\": %d,\n" +
            "  \"executionTimeMs\": %d\n" +
            "}", totalPassed, totalFailed, totalSkipped, totalDuration);
        // 4. Write the JSON payload to a file
        writeReportToFile(outputDirectory, jsonPayload);
    }
    private void writeReportToFile(String outputDirectory, String content) {
        try (FileWriter file = new FileWriter(outputDirectory + "/custom-results.json")) {
            file.write(content);
            System.out.println("Custom JSON Report Generated at: " + outputDirectory + "/custom-results.json");
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

---

## 3. Extracting Individual Test Data

The example above aggregates high-level metrics. But what if you need to extract the exact name and stack trace of every failed test? 

You simply dive deeper into the `ITestContext` object:

```java
// Inside the ISuiteResult loop...
Set<ITestResult> failedTests = tc.getFailedTests().getAllResults();
for (ITestResult failedTest : failedTests) {
    String testName = failedTest.getName();
    String testClass = failedTest.getTestClass().getName();
    Throwable exception = failedTest.getThrowable();
    System.out.println("TEST FAILED: " + testClass + "." + testName);
    System.out.println("REASON: " + exception.getMessage());
}
```

You can append this specific failure data to your custom JSON payload, or use it to programmatically trigger an automated Slack alert!

---

## 4. Registering the IReporter

Just like every other TestNG listener, we must register our Custom Reporter in the `testng.xml` file.

```xml
<!DOCTYPE suite SYSTEM "https://testng.org/testng-1.0.dtd" >
<suite name="Custom Reporting Suite">
    <listeners>
        <listener class-name="com.mycodeyatra.listeners.CustomJsonReporter" />
    </listeners>
    <test name="API and UI Tests">
        <classes>
            <class name="com.mycodeyatra.tests.CheckoutTest" />
        </classes>
    </test>
</suite>
```

When you execute the suite, TestNG will run all the tests normally. But instead of relying on Extent or Allure, it will trigger your custom code at the very end and spit out your proprietary `custom-results.json` file!

## Conclusion

By mastering the `IReporter` interface, you are no longer limited by the constraints of open-source reporting libraries. 

You have learned how to intercept the massive post-execution TestNG data payload, extract exact pass/fail counts, identify specific failing methods and their stack traces, and write that data into any proprietary format your company requires.

**Congratulations!** You have officially and fully completed **Phase 12: Reporting and Analytics**!

Our framework is visually stunning, highly analytical, and structurally flawless. But it is still living entirely on our local machines. 

In our absolute final module of this massive curriculum, **Phase 13: The Cloud & CI/CD**, we will take everything we have built over the past 12 phases and deploy it to the cloud using Docker, Jenkins, and GitHub Actions!
