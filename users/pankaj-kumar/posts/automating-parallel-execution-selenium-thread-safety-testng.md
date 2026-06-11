---
title: Automating Parallel Execution in Selenium WebDriver: Thread-Safety & TestNG
date: 20-Jul-2024
lastUpdated: 11-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [selenium, java, parallel-execution, thread-safety, testng, threadlocal]
category: UI Automation
categories: [UI Automation, Selenium, Java]
excerpt: >-
  Master parallel test execution in Selenium WebDriver. Learn how to configure thread-safe driver sessions using the ThreadLocal pattern and trigger concurrent execution in TestNG.
readTime: 7 min read
---

# Automating Parallel Execution in Selenium WebDriver: Thread-Safety & TestNG

> 📅 **Last Updated:** 11-Jun-2026

As test suites grow, execution times increase dramatically. Running tests sequentially becomes a major bottleneck in CI/CD pipelines, slowing down deployment feedback loops. The solution is **Parallel Execution**—running multiple test scenarios concurrently across separate browser sessions.

However, running UI tests in parallel introduces a major challenge: **Thread Safety**. If multiple threads try to access or share a single WebDriver instance, browser sessions will crash, collide, and hijack each other's execution states. In this tutorial, we will learn how to design thread-safe automation frameworks in Java using the `ThreadLocal` class and TestNG.

---

## 🧭 Thread-Safe Parallel Test Execution Flowchart

The workflow below illustrates how TestNG manages separate worker threads, where each thread initializes and teardowns its own isolated browser session:

![diagram_1](https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/posts/automating-parallel-execution-selenium-thread-safety-testng/images/diagram_1.png)

---

## 🛠️ Step-by-Step Code Walkthrough

To design a thread-safe parallel framework, we utilize Java's `ThreadLocal<WebDriver>` class. This class provides thread-local variables. Each thread that accesses a `ThreadLocal` variable (via its `get()` or `set()` methods) has its own, independently initialized copy of the variable.

Here is the complete implementation of our thread-safe parallel suite:

```java
package com.mycodeyatra.tests;
import io.github.bonigarcia.wdm.WebDriverManager;
import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.chrome.ChromeOptions;
import org.openqa.selenium.support.ui.ExpectedConditions;
import org.openqa.selenium.support.ui.WebDriverWait;
import org.testng.Assert;
import org.testng.annotations.AfterMethod;
import org.testng.annotations.BeforeMethod;
import org.testng.annotations.Test;
import java.time.Duration;
public class ParallelTest {
    // ThreadLocal WebDriver to ensure thread-safety during parallel runs
    private static final ThreadLocal<WebDriver> driverThreadLocal = new ThreadLocal<>();
    public WebDriver getDriver() {
        return driverThreadLocal.get();
    }
    @BeforeMethod
    public void setUp() {
        WebDriverManager.chromedriver().setup();
        ChromeOptions options = new ChromeOptions();
        options.addArguments("--remote-allow-origins=*");
        options.addArguments("--headless=new");
        options.addArguments("--window-size=1920,1080");
        WebDriver driver = new ChromeDriver(options);
        driver.manage().timeouts().implicitlyWait(Duration.ofSeconds(10));
        driverThreadLocal.set(driver);
    }
    @Test
    public void testParallelLogin() {
        WebDriver driver = getDriver();
        long threadId = Thread.currentThread().getId();
        System.out.println("Thread ID: " + threadId + " - Navigating to Login Page");
        driver.get("https://practice.mycodeyatra.com/#/login");
        WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
        WebElement usernameInput = wait.until(ExpectedConditions.presenceOfElementLocated(By.xpath("//input[@data-testid='username']")));
        Assert.assertNotNull(usernameInput);
        System.out.println("Thread ID: " + threadId + " - Login Page Verified successfully!");
    }
    @Test
    public void testParallelForm() {
        WebDriver driver = getDriver();
        long threadId = Thread.currentThread().getId();
        System.out.println("Thread ID: " + threadId + " - Navigating to Form Practice Page");
        driver.get("https://practice.mycodeyatra.com/#/form-practice");
        WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
        WebElement nameInput = wait.until(ExpectedConditions.presenceOfElementLocated(By.xpath("//input[@data-testid='full-name']")));
        Assert.assertNotNull(nameInput);
        System.out.println("Thread ID: " + threadId + " - Form Practice Page Verified successfully!");
    }
    @Test
    public void testParallelSandbox() {
        WebDriver driver = getDriver();
        long threadId = Thread.currentThread().getId();
        System.out.println("Thread ID: " + threadId + " - Navigating to Sandbox Page");
        driver.get("https://practice.mycodeyatra.com/#/sandbox");
        WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
        WebElement sandboxHeader = wait.until(ExpectedConditions.presenceOfElementLocated(By.xpath("//h2")));
        Assert.assertTrue(sandboxHeader.getText().contains("Sandbox"));
        System.out.println("Thread ID: " + threadId + " - Sandbox Page Verified successfully!");
    }
    @AfterMethod
    public void tearDown() {
        WebDriver driver = getDriver();
        long threadId = Thread.currentThread().getId();
        if (driver != null) {
            System.out.println("Thread ID: " + threadId + " - Quitting Driver Session");
            driver.quit();
        }
        driverThreadLocal.remove();
    }
}
```

---

## 💻 Test Execution Logs

Below is the clean console output from executing the test suite in parallel via Maven command-line arguments:

```bash
[INFO] Running com.mycodeyatra.tests.ParallelTest
Thread ID: 19 - Navigating to Sandbox Page
Thread ID: 18 - Navigating to Login Page
Thread ID: 17 - Navigating to Form Practice Page
Thread ID: 17 - Form Practice Page Verified successfully!
Thread ID: 18 - Login Page Verified successfully!
Thread ID: 17 - Quitting Driver Session
Thread ID: 18 - Quitting Driver Session
Thread ID: 19 - Sandbox Page Verified successfully!
Thread ID: 19 - Quitting Driver Session
Tests run: 3, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 8.146 sec
```

Notice how the thread logs (`Thread ID: 17`, `18`, `19`) execute interleaved. All browser sessions start, navigate to their target URLs, perform assertions, and close in parallel!

---

## 📊 Sequential vs Thread-Safe Parallel execution

Comparing execution paradigms:

| Metric / Scenario | Sequential Execution | ThreadLocal Parallel Execution |
| :--- | :--- | :--- |
| **Feedback Loop Speed** | Slow. Time increases linearly with count of tests. | Fast. Limited only by CPU cores and memory threads. |
| **Driver Pollution Risk** | None. Tests share the same session serial step. | None. Each thread acts on an isolated instance. |
| **Hardware Resource Cost** | Minimal (single browser instance run). | High (multiple headless browser engines running). |
| **Implementation Complexity** | Simple. Standard setup fields. | Medium. Requires wrapping driver variables in `ThreadLocal`. |

---

## ⚠️ Common Pitfalls

* **Using a standard static driver instance**: Declaring `public static WebDriver driver;` inside a base class and running tests in parallel is a recipe for disaster. The second thread will override the driver reference initialized by the first thread, leaving the first thread orphaned or causing tests to throw `NullPointerException` or `NoSuchSessionException`.
* **Forgetting `driverThreadLocal.remove()` in teardown**: Calling `driver.quit()` closes the browser, but it does not remove the local reference from the JVM thread mapping. Leaving it can lead to memory leaks in large suites. Always invoke `.remove()` in `@AfterMethod` hooks.
* **Database / Shared State Collisions**: Even if your driver code is thread-safe, if your test cases modify the exact same records in a shared backend database concurrently, they will corrupt each other's assertions. Ensure parallel tests operate on isolated test data records.

---

## 🙋 Frequently Asked Questions

> **Q: How do we trigger parallel tests in TestNG without command line parameters?**
>
> **A:** You can define a XML suite file `testng.xml` and specify parallel attributes inside the `<suite>` tag:
> `<suite name="Parallel Suite" parallel="methods" thread-count="3">`.

> **Q: What is the optimal number of threads to configure for parallel execution?**
>
> **A:** UI automation is highly resource-intensive. As a rule of thumb, spawn no more than **1.5x to 2x** threads than the number of logical CPU cores on your executor machine (e.g., maximum 8-10 threads on a 4-core machine running headlessly). Exceeding this threshold causes resource thrashing and slow test executions.
