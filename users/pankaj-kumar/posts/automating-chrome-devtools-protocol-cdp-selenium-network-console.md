---
title: Automating Chrome DevTools Protocol (CDP) in Selenium: Network and Console Control
date: 19-Jul-2024
lastUpdated: 11-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [selenium, java, cdp, chrome-devtools, network-throttling, geolocation-emulation]
category: UI Automation
categories: [UI Automation, Selenium, Java]
excerpt: >-
  Master native integration with Chrome DevTools Protocol (CDP) in Selenium 4. Learn how to perform network throttling, location emulation, and console log capture using version-independent commands in Java.
readTime: 7 min read
---

# Automating Chrome DevTools Protocol (CDP) in Selenium: Network and Console Control

> 📅 **Last Updated:** 11-Jun-2026

Selenium 4 brought native integration with the Chrome DevTools Protocol (CDP). CDP allows automation testers to inspect, profile, and debug Chromium-based browsers (like Chrome, Edge, and Opera) directly from test scripts.

Historically, features like network request interception, geo-location spoofing, console log capture, and simulating slow connections required external proxies (like BrowserMob Proxy). With Selenium 4's CDP integration, these capabilities are native, fast, and robust. In this guide, we'll demonstrate how to perform network throttling, location emulation, and console log capture using version-independent commands.

---

## 🧭 Chrome DevTools Protocol Architecture

The flow diagram below displays how Selenium communicates natively via the DevTools WebSocket connection to trigger runtime browser emulation:

![diagram_1](https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/posts/automating-chrome-devtools-protocol-cdp-selenium-network-console/images/diagram_1.png)

---

## 🛠️ Step-by-Step Code Walkthrough

Rather than importing version-specific Java packages (like `org.openqa.selenium.devtools.v120`), which break when the browser version updates, we utilize the version-independent `ChromiumDriver.executeCdpCommand(String method, Map<String, Object> params)` API. This approach ensures your test suite remains robust across browser upgrades.

Here is the complete implementation of our CDP test suite:

```java
package com.mycodeyatra.tests;
import io.github.bonigarcia.wdm.WebDriverManager;
import org.openqa.selenium.JavascriptExecutor;
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
import java.util.HashMap;
import java.util.Map;
import java.util.logging.Level;
public class CdpTest {
    private ChromeDriver driver;
    @BeforeMethod
    public void setUp() {
        WebDriverManager.chromedriver().setup();
        ChromeOptions options = new ChromeOptions();
        options.addArguments("--remote-allow-origins=*");
        options.addArguments("--headless=new");
        // Enable Browser console logging preferences
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
        // Inject custom console messages via JavascriptExecutor to simulate application behavior
        System.out.println("Injecting custom console warning and error messages...");
        JavascriptExecutor js = (JavascriptExecutor) driver;
        js.executeScript("console.warn('CDP Test: This is a custom warning message');");
        js.executeScript("console.error('CDP Test: This is a custom error message');");
        // Fetch browser console log entries
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
        // Assert that the injected logs were successfully captured
        Assert.assertTrue(foundWarning, "Injected console warning was not captured.");
        Assert.assertTrue(foundError, "Injected console error was not captured.");
        System.out.println("CDP console logs validation complete!");
    }
    @Test
    public void testNetworkThrottlingEmulation() {
        System.out.println("Enabling network throttling via version-independent executeCdpCommand...");
        Map<String, Object> networkConditions = new HashMap<>();
        networkConditions.put("offline", false);
        networkConditions.put("latency", 100); // 100ms latency
        networkConditions.put("downloadThroughput", 50 * 1024); // 50 KB/s download speed
        networkConditions.put("uploadThroughput", 20 * 1024); // 20 KB/s upload speed
        // Execute CDP command to emulate network conditions
        driver.executeCdpCommand("Network.emulateNetworkConditions", networkConditions);
        long startTime = System.currentTimeMillis();
        System.out.println("Navigating to live login URL on throttled connection...");
        driver.get("https://practice.mycodeyatra.com/#/login");
        long duration = System.currentTimeMillis() - startTime;
        System.out.println("Page loaded in " + duration + " ms.");
        // Verify we navigated successfully
        Assert.assertTrue(driver.getCurrentUrl().contains("login"), "Failed to load Login page under throttled network!");
    }
    @Test
    public void testGeolocationEmulation() {
        System.out.println("Emulating Tokyo, Japan location coordinates via CDP...");
        Map<String, Object> coordinates = new HashMap<>();
        coordinates.put("latitude", 35.6762);
        coordinates.put("longitude", 139.6503);
        coordinates.put("accuracy", 1);
        // Execute CDP command to emulate geolocation
        driver.executeCdpCommand("Emulation.setGeolocationOverride", coordinates);
        driver.get("https://practice.mycodeyatra.com/");
        // Fetch geolocation coordinates from browser window context to verify emulation
        JavascriptExecutor js = (JavascriptExecutor) driver;
        js.executeScript("navigator.geolocation.getCurrentPosition(function(position) { " +
                "console.log('Lat: ' + position.coords.latitude + ' Lon: ' + position.coords.longitude); " +
                "});");
        System.out.println("Tokyo Geolocation Emulation complete!");
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

Below is the clean console output from running the CDP test cases:

```bash
[INFO] Running com.mycodeyatra.tests.CdpTest
Navigating to: https://practice.mycodeyatra.com/
Injecting custom console warning and error messages...
Fetching console logs...
[WARNING] console-api 2:32 "CDP Test: This is a custom warning message"
[SEVERE] console-api 2:32 "CDP Test: This is a custom error message"
CDP console logs validation complete!
Quitting driver session...
Emulating Tokyo, Japan location coordinates via CDP...
Tokyo Geolocation Emulation complete!
Quitting driver session...
Enabling network throttling via version-independent executeCdpCommand...
Navigating to live login URL on throttled connection...
Page loaded in 589 ms.
Quitting driver session...
Tests run: 3, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 9.098 sec
```

---

## 📊 Comparison of CDP Integration Approaches

There are two primary ways to run Chrome DevTools commands in Selenium:

| Comparison Metric | DevTools Wrapper Class | `executeCdpCommand` |
| :--- | :--- | :--- |
| **API Type** | Strongly-typed Java wrapper methods | Map/Dictionary based JSON commands |
| **Maintenance** | ❌ High. Requires compilation fixes when versioned imports break on browser updates. | ✅ Low. Strings and maps are backward and forward-compatible. |
| **Browser Compatibility** | Chrome and Chromium Edge only. | Chrome and Chromium Edge only. |
| **Autocompletion** | Easy IDE auto-completion suggestions. | Needs consulting Chrome DevTools documentation for method names. |

---

## ⚠️ Common Pitfalls

* **Version Mismatch Errors**: If you import `org.openqa.selenium.devtools.v123.network.Network` but run tests on Chrome v125, your tests will fail immediately on startup. Standardize on `executeCdpCommand` to bypass version boundaries.
* **Not resetting emulation overrides**: Spoofing variables like Geolocation coordinates persists across the lifetime of the WebDriver instance. If subsequent tests rely on local locations, you must clean overrides using `driver.executeCdpCommand("Emulation.clearGeolocationOverride", new HashMap<>());`.
* **Throttling limits in CI/CD pipeline**: Emulating slow networks directly impacts page loading times. Headless CI/CD containers often have tight runner limits. Make sure to adjust implicit or explicit timeouts globally when throttling is enabled.

---

## 🙋 Frequently Asked Questions

> **Q: Can we intercept and modify network headers or mock API responses via CDP?**
>
> **A:** Yes. You can execute `Network.setRequestInterception` or `Fetch.enable` commands via CDP to pause requests, modify request headers on the fly, or mock responses with custom status codes and payloads.

> **Q: Does CDP work for Firefox or Safari drivers?**
>
> **A:** CDP is specific to Chromium-based browsers. Firefox has a partial implementation of CDP, but it is not natively supported. To perform cross-browser intercepts, W3C BiDi (Bidirectional WebDriver protocol) is being integrated into modern Selenium versions.
