---
title: Custom TestNG Listeners: Advanced Reporting and Retry Logic
date: 08-Aug-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [selenium, java, testng, listeners, automation, framework]
category: UI Automation
categories: [UI Automation, Selenium, Java]
excerpt: >-
  Build custom TestNG listeners to automate screenshot capture on failure, log errors cleanly, and retry flaky tests using IRetryAnalyzer and IAnnotationTransformer.
readTime: 5 min read
---

# Custom TestNG Listeners: Advanced Reporting and Retry Logic

As your test suite grows, manually digging through logs to figure out why a test failed becomes incredibly tedious. You need a system that automatically reacts to test events—capturing screenshots on failure, logging errors, and even retrying flaky tests automatically. 

In this article, we dive into building **Custom TestNG Listeners**. By implementing `ITestListener` and `IRetryAnalyzer`, we can inject powerful, automated behaviors into our Selenium Java framework without polluting our test classes.

---

## 1. The Power of `ITestListener`

TestNG provides the `ITestListener` interface, which exposes hook methods triggered during different phases of a test's lifecycle. 

Let's build a `TestExecutionListener` that hooks into `onTestFailure` to automatically capture a screenshot and log the error stack trace.

```java
package com.mycodeyatra.listeners;
 
import org.openqa.selenium.OutputType;
import org.openqa.selenium.TakesScreenshot;
import org.openqa.selenium.WebDriver;
import org.testng.ITestListener;
import org.testng.ITestResult;
import org.apache.logging.log4j.LogManager;
import org.apache.logging.log4j.Logger;
import java.io.File;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;
 
public class TestExecutionListener implements ITestListener {
 
    private static final Logger logger = LogManager.getLogger(TestExecutionListener.class);
 
    @Override
    public void onTestFailure(ITestResult result) {
        logger.error("Test FAILED: {}", result.getName());
        logger.error("Reason: {}", result.getThrowable().getMessage());
 
        // Retrieve the WebDriver instance from your BaseTest or Spring Context
        Object testClass = result.getInstance();
        WebDriver driver = ((SpringBaseTest) testClass).getDriver();
 
        if (driver != null) {
            captureScreenshot(driver, result.getName());
        }
    }
 
    private void captureScreenshot(WebDriver driver, String testName) {
        try {
            File src = ((TakesScreenshot) driver).getScreenshotAs(OutputType.FILE);
            Path dest = Paths.get("target/screenshots/" + testName + ".png");
            Files.createDirectories(dest.getParent());
            Files.copy(src.toPath(), dest);
            logger.info("Screenshot saved to: {}", dest.toString());
        } catch (Exception e) {
            logger.error("Failed to capture screenshot: ", e);
        }
    }
 
    @Override
    public void onTestStart(ITestResult result) {
        logger.info("Starting test: {}", result.getName());
    }
 
    @Override
    public void onTestSuccess(ITestResult result) {
        logger.info("Test PASSED: {}", result.getName());
    }
}
```

---

## 2. Automating Retries with `IRetryAnalyzer`

Flakiness is the enemy of CI/CD. Network blips or rendering delays can cause a test to fail intermittently. Instead of marking the build as failed immediately, we can use `IRetryAnalyzer` to retry the test a specified number of times before giving up.

```java
package com.mycodeyatra.listeners;
 
import org.testng.IRetryAnalyzer;
import org.testng.ITestResult;
import org.apache.logging.log4j.LogManager;
import org.apache.logging.log4j.Logger;
 
public class RetryAnalyzer implements IRetryAnalyzer {
 
    private static final Logger logger = LogManager.getLogger(RetryAnalyzer.class);
 
    private int currentTry = 0;
    private static final int MAX_RETRIES = 2; // Retry failed tests up to 2 times
 
    @Override
    public boolean retry(ITestResult result) {
        if (currentTry < MAX_RETRIES) {
            logger.warn("Retrying test '{}' - Attempt {} of {}", 
                        result.getName(), currentTry + 1, MAX_RETRIES);
            currentTry++;
            return true; // Tells TestNG to retry the test
        }
        return false; // Stop retrying
    }
}
```

---

## 3. Auto-Applying the Retry Logic

By default, you have to add `@Test(retryAnalyzer = RetryAnalyzer.class)` to every single test method. This is tedious and prone to human error. Instead, we can use `IAnnotationTransformer` to inject the retry logic into *all* tests globally.

```java
package com.mycodeyatra.listeners;
 
import org.testng.IAnnotationTransformer;
import org.testng.annotations.ITestAnnotation;
import java.lang.reflect.Constructor;
import java.lang.reflect.Method;
 
public class AnnotationTransformer implements IAnnotationTransformer {
 
    @Override
    public void transform(ITestAnnotation annotation, Class testClass, 
                          Constructor testConstructor, Method testMethod) {
 
        // Globally assign the RetryAnalyzer to every test
        annotation.setRetryAnalyzer(RetryAnalyzer.class);
    }
}
```

---

## 4. Hooking It All Together in `testng.xml`

To activate these listeners across your entire test suite, register them in your `testng.xml` suite file.

```xml
<!DOCTYPE suite SYSTEM "https://testng.org/testng-1.0.dtd">
<suite name="Enterprise Automation Suite">
 
    <listeners>
        <!-- Handles Screenshots & Logging -->
        <listener class-name="com.mycodeyatra.listeners.TestExecutionListener"/>
        <!-- Globally Injects Retry Logic -->
        <listener class-name="com.mycodeyatra.listeners.AnnotationTransformer"/>
    </listeners>
 
    <test name="Login Regression">
        <classes>
            <class name="com.mycodeyatra.tests.LoginTest"/>
        </classes>
    </test>
</suite>
```

---

## System Architecture

Here is how the Listeners orchestrate the test execution lifecycle:

![diagram_1](https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/posts/custom-testng-listeners-advanced-reporting-and-retry-logic/images/diagram_1.png)

## Advantages of Using Listeners

1. **Separation of Concerns:** Test classes focus strictly on business logic and assertions. They do not handle screenshot logic or retry counters.
2. **Global Configuration:** An `IAnnotationTransformer` ensures no test is ever left behind when applying retry logic.
3. **Rich Reporting:** The exact point of failure is immediately captured with screenshots and logs, drastically reducing debugging time.

In our next and final framework phase, we will tackle **Parallel Execution with ThreadLocal**, taking your robust framework and scaling it to run dozens of tests concurrently!
