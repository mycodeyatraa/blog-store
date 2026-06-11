---
title: Chrome DevTools Protocol (CDP) and Browser Logging in Selenium
date: 09-Jul-2024
lastUpdated: 11-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [selenium, java, cdp, chrome-devtools, logging]
category: UI Automation
categories: [UI Automation, Selenium, Java]
excerpt: >-
  Learn how to leverage the Chrome DevTools Protocol (CDP) and collect browser console logs using Selenium WebDriver in Java, featuring a complete step-by-step TestNG automation guide.
readTime: 7 min read
---

# Chrome DevTools Protocol (CDP) and Browser Logging in Selenium

> 📅 **Last Updated:** 11-Jun-2026

Modern web applications rely heavily on client-side JavaScript execution. When tests fail, the cause is often hidden inside browser console errors, failed network requests, or security warning logs. Capturing these logs during automation is critical for debugging flaky test suites.

In this guide, we will explore how to interface with Chrome DevTools Protocol (CDP) and capture browser console logs using Selenium WebDriver with Java and TestNG.

---

## 🛰️ WebDriver vs. CDP Communication Architecture

To understand how Selenium captures console logs and interacts with browser internals, it is helpful to look at the communication architecture. 

In traditional Selenium 3, communication was strictly restricted to HTTP commands over the JSON Wire Protocol. Selenium 4 introduces native bidirectional support using WebSocket connections, allowing the driver client to bypass the HTTP API and interface directly with Chrome DevTools Protocol (CDP) domains.

![diagram_1](https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/posts/chrome-devtools-protocol-cdp-browser-logging-selenium/images/diagram_1.png)

---

## 🛠️ Step-by-Step Code Walkthrough

To capture console errors, we need to instruct ChromeDriver to collect logs before the browser session starts. This is done by configuring `LoggingPreferences` inside `ChromeOptions`.

Here is the complete implementation showing how to capture, filter, and assert browser console errors and warnings using TestNG:

```java
package com.mycodeyatra.tests;
import io.github.bonigarcia.wdm.WebDriverManager;
import org.openqa.selenium.JavascriptExecutor;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.chrome.ChromeOptions;
import org.openqa.selenium.logging.LogEntries;
import org.openqa.selenium.logging.LogEntry;
import org.openqa.selenium.logging.LogType;
import org.openqa.selenium.logging.LoggingPreferences;
import org.testng.Assert;
import org.testng.annotations.AfterMethod;
import org.testng.annotations.BeforeMethod;
import org.testng.annotations.Test;
import java.time.Duration;
import java.util.logging.Level;
public class CdpTest {
    private WebDriver driver;
    @BeforeMethod
    public void setUp() {
        WebDriverManager.chromedriver().setup();
        ChromeOptions options = new ChromeOptions();
        options.addArguments("--remote-allow-origins=*");
        options.addArguments("--headless=new"); // Keep execution clean in CI pipelines
        // 1. Enable Browser console logging preferences
        LoggingPreferences logPrefs = new LoggingPreferences();
        logPrefs.enable(LogType.BROWSER, Level.ALL);
        options.setCapability(ChromeOptions.LOGGING_PREFS, logPrefs);
        driver = new ChromeDriver(options);
        driver.manage().window().maximize();
        driver.manage().timeouts().implicitlyWait(Duration.ofSeconds(10));
    }
    @Test
    public void testBrowserConsoleLogs() {
        System.out.println("Navigating to: https://practice.mycodeyatra.com/");
        driver.get("https://practice.mycodeyatra.com/");
        // 2. Inject custom console messages via JavaScript to simulate app errors
        System.out.println("Injecting custom console warning and error messages...");
        JavascriptExecutor js = (JavascriptExecutor) driver;
        js.executeScript("console.warn('CDP Test: This is a custom warning message');");
        js.executeScript("console.error('CDP Test: This is a custom error message');");
        // 3. Fetch browser console log entries
        System.out.println("Fetching console logs...");
        LogEntries logEntries = driver.manage().logs().get(LogType.BROWSER);
        boolean foundWarning = false;
        boolean foundError = false;
        for (LogEntry entry : logEntries) {
            System.out.println("[" + entry.getLevel() + "] " + entry.getMessage());
            if (entry.getMessage().contains("This is a custom warning message")) {
                foundWarning = true;
            }
            if (entry.getMessage().contains("This is a custom error message")) {
                foundError = true;
            }
        }
        // 4. Assert that the logs were successfully captured
        Assert.assertTrue(foundWarning, "Injected console warning was not captured.");
        Assert.assertTrue(foundError, "Injected console error was not captured.");
        System.out.println("CDP console logs validation complete!");
    }
    @AfterMethod
    public void tearDown() {
        if (driver != null) {
            System.out.println("Quitting driver session...");
            driver.quit();
        }
    }
}
```

---

## 💻 Test Execution Logs

Here are the console outputs showing our compilation, setup, injection, and execution capturing logs cleanly:

```bash
[INFO] Running com.mycodeyatra.tests.CdpTest
Navigating to: https://practice.mycodeyatra.com/
Injecting custom console warning and error messages...
Fetching console logs...
[WARNING] https://practice.mycodeyatra.com/ 0:49 "CDP Test: This is a custom warning message"
[SEVERE] https://practice.mycodeyatra.com/ 0:50 "CDP Test: This is a custom error message"
CDP console logs validation complete!
Quitting driver session...
Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 5.378 sec
```

---

## 📊 Logging Configurations

Selenium supports multiple driver log domains. Choosing the correct type ensures your framework is not bloated with unrelated telemetry data.

| Log Type Domain | Captured Data Source | Recommended Use Cases |
| :--- | :--- | :--- |
| `LogType.BROWSER` | JavaScript errors, console warnings, deprecations | Client-side scripting bug detection |
| `LogType.CLIENT` | Local webdriver client logging | Debugging local Java-side configurations |
| `LogType.DRIVER` | Chrome/Gecko driver server daemon logs | Locating selenium driver binary crashes |
| `LogType.PERFORMANCE` | Network requests, frame rates, memory footprints | Client-side performance auditing |

---

## ⚠️ Common Pitfalls

* **CDP Client Version Lockups**: Directly importing `org.openqa.selenium.devtools.v125.log.Log` locks your test code to Chrome version 125. If the browser auto-updates (e.g. to version 149), compilation will crash. Use the version-independent `LogType.BROWSER` unless complex CDP actions are required.
* **Flaky Cleared Logs**: Log queues are automatically cleared once `driver.manage().logs().get(LogType.BROWSER)` is called. Calling this method twice in a single test block will return an empty list on the second invocation.
* **Driver Quitting Race Conditions**: Always fetch your log buffer before calling `driver.quit()`. Once the browser socket closes, console log memory queues are destroyed.

---

## 🙋 Frequently Asked Questions

> **Q: How does headless execution affect browser console logging?**
>
> **A:** Headless browsers execute the same rendering engines (like V8 in Chromium) as headful instances. Logging is completely unaffected, making console error capturing an excellent choice for headless CI/CD runs.

> **Q: Can we capture network traffic status codes via browser console logs?**
>
> **A:** Basic failed requests (like `404 Not Found` or `500 Internal Server Error`) are output to console logs by default. However, to capture successful `200 OK` calls or response headers, you must use CDP Network interception options (`Network.enable`) instead.
