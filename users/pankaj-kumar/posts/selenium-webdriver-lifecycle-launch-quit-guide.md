---
title: Mastering Selenium WebDriver Lifecycle: Launch, Quit & Resource Management
date: 05-Jul-2024
lastUpdated: 10-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [selenium, java, architecture, best-practices]
category: UI Automation
categories: [UI Automation, Selenium, Java]
excerpt: >-
  Improper resource management is the #1 cause of flaky tests and crashed CI/CD runner nodes. Learn how to manage the lifecycle of your Selenium WebDriver instances, prevent memory leaks, and cleanly dispose of driver sessions.
readTime: 6 min read
---

# Mastering Selenium WebDriver Lifecycle: Launch, Quit & Resource Management

> 📅 **Last Updated:** 10-Jun-2026

Improper resource management is the number one cause of flaky automation tests and crashed CI/CD build agents. When test suites leave orphaned browser processes behind, system memory quickly exhausts, leading to cascading test failures. 

Understanding how to initialize, manage, and cleanly dispose of browser sessions is a fundamental skill for test automation engineers. In this guide, we deep dive into the WebDriver lifecycle, dissect the functional difference between `close()` and `quit()`, and look at modern Java patterns to guarantee resource cleanup.

---

## 🔄 The Life Cycle of a WebDriver Session

A WebDriver session operates in a strict three-phase lifecycle: **Initialization**, **Active Execution**, and **Teardown**.

![diagram_1](https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/posts/selenium-webdriver-lifecycle-launch-quit-guide/images/diagram_1.png)

When you initialize a driver (e.g., `WebDriver driver = new ChromeDriver()`), the language bindings send a request to launch the driver server process (like `chromedriver.exe`). The driver server then starts a browser session. That process must be destroyed at the end of the test.

---

## ⚔️ The Great Debate: driver.close() vs. driver.quit()

One of the most common mistakes in test automation is using `close()` when you should be using `quit()`. While both methods tear down parts of the browser state, they handle operating system resources very differently.

### Differences at a Glance

| Aspect | `driver.close()` | `driver.quit()` |
| :--- | :--- | :--- |
| **Target** | Closes the current active window or tab. | Closes all associated windows, tabs, and sessions. |
| **Session Status** | Session remains active if other tabs are open. | Destroys the entire WebDriver session. |
| **Driver Server Process** | Keeps the helper driver process (`chromedriver.exe`) running. | Kills the helper driver process and terminates the background OS daemon. |
| **Use Case** | Multi-window navigation handling. | Final teardown at the end of execution. |

Here is what happens when you miss `driver.quit()`:

![diagram_2](https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/posts/selenium-webdriver-lifecycle-launch-quit-guide/images/diagram_2.png)

> [!WARNING]
> Calling `driver.close()` as your final teardown step will leave `chromedriver` or `geckodriver` processes running silently in the background. In CI/CD runners, this will quickly consume all system RAM and crash your pipeline.

---

## 🛠️ Java Best Practices for Resource Management

To prevent resource leaks, you must ensure that teardown code executes regardless of whether the test passes, fails, or throws an unhandled exception.

### 1. The Standard TestNG Hook Pattern

The standard industry approach is placing `driver.quit()` in a `@AfterMethod` or `@AfterClass` hook block. TestNG guarantees these execution hooks run even if assertions fail during the test.

```java
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.chrome.ChromeOptions;
import org.testng.annotations.AfterMethod;
import org.testng.annotations.BeforeMethod;
import org.testng.annotations.Test;
public class LifecycleTest {
    private WebDriver driver;
    @BeforeMethod
    public void setup() {
        ChromeOptions options = new ChromeOptions();
        options.addArguments("--headless=new");
        driver = new ChromeDriver(options);
    }
    @Test
    public void executeTest() {
        driver.get("https://practice.mycodeyatra.com/");
        // Execution steps go here
    }
    @AfterMethod
    public void teardown() {
        if (driver != null) {
            driver.quit(); // Releases resources cleanly
        }
    }
}
```

### 2. The Try-With-Resources Pattern (Modern Java)

In modern Selenium 4, the `WebDriver` interface extends `AutoCloseable`. This allows you to use Java's **try-with-resources** statement, which handles resource cleanup automatically at the end of the block.

```java
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.chrome.ChromeOptions;
public class ModernLifecycle {
    public static void main(String[] args) {
        ChromeOptions options = new ChromeOptions();
        options.addArguments("--headless=new");
        // Try-with-resources auto-invokes driver.quit() on termination
        try (WebDriver driver = new ChromeDriver(options)) {
            driver.get("https://practice.mycodeyatra.com/");
            System.out.println("Page loaded: " + driver.getTitle());
        } // Driver automatically destroyed here
    }
}
```

---

## 💡 Summary: Lifecycle Best Practices

* **Always Quit**: Make `driver.quit()` the default closing command of your framework.
* **Null Checks**: Guard your teardown blocks with `if (driver != null)` to avoid throwing `NullPointerException` during teardown if initialization failed.
* **Avoid Hard Kills**: Try to let the framework naturally quit instead of executing `taskkill` scripts, which can corrupt local browser profile caches.
