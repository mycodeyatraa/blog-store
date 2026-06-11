---
title: TestNG Lifecycle & Listeners in Selenium: Mastering Test Hooks and Event Interception
date: 27-Jul-2024
lastUpdated: 11-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [selenium, java, testng, listeners, lifecycle, retry-analyzer]
category: UI Automation
categories: [UI Automation, Selenium, Java]
excerpt: >-
  Master TestNG lifecycle annotations (@Before/@After) and build a custom ITestListener and IRetryAnalyzer in Java Selenium to intercept every test event and handle flaky tests automatically.
readTime: 6 min read
---

# TestNG Lifecycle & Listeners in Selenium: Mastering Test Hooks and Event Interception

Understanding TestNG's lifecycle is the key to building **observable, maintainable, and robust** Selenium test suites. In this blog, we go beyond the basics — we implement a custom `ITestListener` that intercepts every test lifecycle event, and an `IRetryAnalyzer` that automatically re-runs flaky tests without any manual intervention.

---

## What You Will Build

| Component | Interface | Purpose |
|---|---|---|
| `CustomTestListener` | `ITestListener` | Log every test lifecycle event automatically |
| `RetryAnalyzer` | `IRetryAnalyzer` | Auto-retry failing tests up to N times |
| `LifecycleTest` | TestNG test class | Demonstrates the complete @Before/@After hook chain |

---

## The TestNG Lifecycle — In Order

The diagram below shows the exact execution order of TestNG annotations for a single test class:

![diagram_1](https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/posts/testng-lifecycle-listeners-selenium-java/images/diagram_1.png)

---

## Project Structure

We create two listener classes under `src/main/java/com/mycodeyatra/listeners/` and update the existing `LifecycleTest`:

```
mcyt-sel-java/
 src/
  main/java/com/mycodeyatra/
   listeners/
    CustomTestListener.java   <-- ITestListener implementation
    RetryAnalyzer.java        <-- IRetryAnalyzer implementation
  test/java/com/mycodeyatra/
   tests/
    LifecycleTest.java        <-- @Listeners-annotated test class
```

---

## Step-by-Step Code Walkthrough

### 1. Custom ITestListener (CustomTestListener.java)

The `ITestListener` interface gives you a callback for every significant lifecycle event — suite start/finish, and each test's start, pass, fail, and skip.

```java
package com.mycodeyatra.listeners;
import org.testng.ITestContext;
import org.testng.ITestListener;
import org.testng.ITestResult;
public class CustomTestListener implements ITestListener {
 @Override
 public void onStart(ITestContext context) {
  System.out.println("====================================");
  System.out.println("[Listener] Suite Starting: " + context.getName());
  System.out.println("====================================");
 }
 @Override
 public void onFinish(ITestContext context) {
  System.out.println("====================================");
  System.out.println("[Listener] Suite Finished: " + context.getName());
  System.out.println("  Passed  : " + context.getPassedTests().size());
  System.out.println("  Failed  : " + context.getFailedTests().size());
  System.out.println("  Skipped : " + context.getSkippedTests().size());
  System.out.println("====================================");
 }
 @Override
 public void onTestStart(ITestResult result) {
  System.out.println("[Listener] TEST STARTED  -> " + result.getName());
 }
 @Override
 public void onTestSuccess(ITestResult result) {
  System.out.println("[Listener] TEST PASSED   -> " + result.getName());
 }
 @Override
 public void onTestFailure(ITestResult result) {
  System.out.println("[Listener] TEST FAILED   -> " + result.getName());
  System.out.println("[Listener] Failure cause : " + result.getThrowable().getMessage());
 }
 @Override
 public void onTestSkipped(ITestResult result) {
  System.out.println("[Listener] TEST SKIPPED  -> " + result.getName());
 }
 @Override
 public void onTestFailedButWithinSuccessPercentage(ITestResult result) {
  System.out.println("[Listener] TEST PARTIALLY PASSED -> " + result.getName());
 }
}
```

### 2. Retry Analyzer (RetryAnalyzer.java)

The `IRetryAnalyzer` interface exposes a single `retry()` method. Return `true` to re-run the test, `false` to mark it as a final failure.

```java
package com.mycodeyatra.listeners;
import org.testng.IRetryAnalyzer;
import org.testng.ITestResult;
public class RetryAnalyzer implements IRetryAnalyzer {
 private int retryCount = 0;
 private static final int MAX_RETRY_COUNT = 2;
 @Override
 public boolean retry(ITestResult result) {
  if (retryCount < MAX_RETRY_COUNT) {
   retryCount++;
   System.out.println("[RetryAnalyzer] Retrying test: " + result.getName()
    + " | Attempt: " + retryCount + " of " + MAX_RETRY_COUNT);
   return true;
  }
  return false;
 }
}
```

### 3. Test Class with Full Lifecycle (LifecycleTest.java)

Wire the listener via `@Listeners` and attach `RetryAnalyzer` per test via `retryAnalyzer` attribute:

```java
package com.mycodeyatra.tests;
import com.mycodeyatra.driver.DriverFactory;
import com.mycodeyatra.listeners.CustomTestListener;
import com.mycodeyatra.listeners.RetryAnalyzer;
import org.openqa.selenium.WebDriver;
import org.testng.Assert;
import org.testng.annotations.AfterMethod;
import org.testng.annotations.BeforeClass;
import org.testng.annotations.BeforeMethod;
import org.testng.annotations.Listeners;
import org.testng.annotations.Optional;
import org.testng.annotations.Parameters;
import org.testng.annotations.Test;
@Listeners(CustomTestListener.class)
public class LifecycleTest {
 private WebDriver driver;
 @BeforeClass
 public void suiteSetup() {
  System.out.println("[Lifecycle] @BeforeClass  -> Suite-level setup (runs once per class)");
 }
 @BeforeMethod
 @Parameters("browser")
 public void setup(@Optional("chrome") String browser) {
  System.out.println("[Lifecycle] @BeforeMethod -> Initialising " + browser + " browser...");
  driver = DriverFactory.initDriver(browser);
  driver.manage().window().maximize();
 }
 @Test(description = "Verify the practice site title loads correctly")
 public void verifyPageTitle() {
  System.out.println("[Lifecycle] @Test         -> verifyPageTitle running...");
  driver.get("https://practice.mycodeyatra.com/");
  String title = driver.getTitle();
  System.out.println("[Lifecycle] Page Title: " + title);
  Assert.assertFalse(title.isEmpty(), "Page title should not be empty");
 }
 @Test(description = "Simulate a flaky test with RetryAnalyzer",
  retryAnalyzer = RetryAnalyzer.class)
 public void simulateFlakyTest() {
  System.out.println("[Lifecycle] @Test         -> simulateFlakyTest running...");
  driver.get("https://practice.mycodeyatra.com/");
  String url = driver.getCurrentUrl();
  Assert.assertTrue(url.contains("practice.mycodeyatra.com"),
   "URL should contain practice.mycodeyatra.com");
 }
 @AfterMethod
 public void teardown() {
  System.out.println("[Lifecycle] @AfterMethod  -> Closing browser session...");
  DriverFactory.quitDriver();
 }
}
```

---

## Registering Listeners via testng.xml

You can also register listeners globally in `testng.xml` instead of using `@Listeners`:

```xml
<!DOCTYPE suite SYSTEM "https://testng.org/testng-1.0.dtd">
<suite name="Lifecycle Suite" parallel="none">
 <listeners>
  <listener class-name="com.mycodeyatra.listeners.CustomTestListener"/>
 </listeners>
 <test name="Lifecycle Tests">
  <parameter name="browser" value="chrome"/>
  <classes>
   <class name="com.mycodeyatra.tests.LifecycleTest"/>
  </classes>
 </test>
</suite>
```

---

## Console Output — What to Expect

When you run `LifecycleTest`, the console output follows this exact order:

```
====================================
[Listener] Suite Starting: Lifecycle Tests
====================================
[Lifecycle] @BeforeClass  -> Suite-level setup (runs once per class)
[Lifecycle] @BeforeMethod -> Initialising chrome browser...
[Listener] TEST STARTED  -> verifyPageTitle
[Lifecycle] @Test         -> verifyPageTitle running...
[Lifecycle] Page Title: MyCodeYatra Practice Site
[Listener] TEST PASSED   -> verifyPageTitle
[Lifecycle] @AfterMethod  -> Closing browser session...
[Lifecycle] @BeforeMethod -> Initialising chrome browser...
[Listener] TEST STARTED  -> simulateFlakyTest
[Lifecycle] @Test         -> simulateFlakyTest running...
[Listener] TEST PASSED   -> simulateFlakyTest
[Lifecycle] @AfterMethod  -> Closing browser session...
====================================
[Listener] Suite Finished: Lifecycle Tests
  Passed  : 2
  Failed  : 0
  Skipped : 0
====================================
```

---

## Key Takeaways

1. **Strict execution order**: `@BeforeSuite -> @BeforeClass -> @BeforeMethod -> @Test -> @AfterMethod -> @AfterClass -> @AfterSuite` — TestNG guarantees this order.
2. **Listeners decouple cross-cutting concerns**: Logging, reporting, and screenshot capture belong in listeners — not inside test classes.
3. **IRetryAnalyzer handles flakiness gracefully**: Instead of manual re-runs, annotate flaky tests with `retryAnalyzer` and let TestNG handle it.
4. **Global vs. local listener registration**: Use `@Listeners` for class-scoped listeners, `testng.xml` for suite-scoped global listeners.
