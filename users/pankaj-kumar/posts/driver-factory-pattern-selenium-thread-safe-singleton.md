---
title: Driver Factory Pattern in Selenium: Designing a Thread-Safe Singleton
date: 24-Jul-2024
lastUpdated: 11-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [selenium, java, driver-factory, design-patterns, singleton, thread-local, parallel-testing]
category: UI Automation
categories: [UI Automation, Selenium, Java]
excerpt: >-
  Master the Driver Factory Pattern in Selenium WebDriver. Learn how to implement a thread-safe singleton DriverFactory using ThreadLocal, configure cross-browser headless runs, and ensure safe lifecycle cleanup.
readTime: 6 min read
---

# Driver Factory Pattern in Selenium: Designing a Thread-Safe Singleton

> 📅 **Last Updated:** 11-Jun-2026

In professional test automation suites, managing the lifecycle of the browser driver is critical. Standard instantiations where the driver is passed down through constructors or kept as a simple static field often fail when running tests in parallel. Different threads override each other's browser sessions, causing tests to crash, mix up execution state, or terminate prematurely.

The **Driver Factory Pattern** combined with a thread-safe singleton design solves this by encapsulating browser initialization and isolating each browser session to its own execution thread using Java's `ThreadLocal`.

---

## 🧭 Driver Factory Architecture

The diagram below shows how parallel test threads fetch isolated, browser-specific WebDriver instances from the thread-local driver registry:

![diagram_1](https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/posts/driver-factory-pattern-selenium-thread-safe-singleton/images/diagram_1.png)

---

## 🛠️ Step-by-Step Code Walkthrough

We create our singleton thread-safe driver factory in the main source package `src/main/java/com/mycodeyatra/driver/` and test it under `src/test/java/com/mycodeyatra/tests/`.

### 1. The Thread-Safe Factory Class (`DriverFactory.java`)
This class provides static accessors to initialize, fetch, and close the driver. By using `ThreadLocal<WebDriver>`, we ensure that calling `getDriver()` from any thread returns the unique instance allocated to that specific thread.

```java
package com.mycodeyatra.driver;
import io.github.bonigarcia.wdm.WebDriverManager;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.chrome.ChromeOptions;
import org.openqa.selenium.edge.EdgeDriver;
import org.openqa.selenium.edge.EdgeOptions;
import org.openqa.selenium.firefox.FirefoxDriver;
import org.openqa.selenium.firefox.FirefoxOptions;
import java.time.Duration;
/**
 * DriverFactory is a thread-safe singleton class that manages WebDriver instances
 * using ThreadLocal to support parallel and clean test execution.
 */
public class DriverFactory {
    private static final ThreadLocal<WebDriver> driverThreadLocal = new ThreadLocal<>();
    // Private constructor to prevent direct instantiation
    private DriverFactory() {}
    /**
     * Initializes the driver session for the current thread based on the specified browser.
     * Defaults to Chrome if no browser is specified.
     * 
     * @param browser The browser name ("chrome", "firefox", "edge")
     * @return The initialized WebDriver instance
     */
    public static WebDriver initDriver(String browser) {
        if (driverThreadLocal.get() == null) {
            WebDriver driver;
            String browserName = (browser != null) ? browser.trim().toLowerCase() : "chrome";
            System.out.println("[DriverFactory] Initializing WebDriver for browser: " + browserName);
            switch (browserName) {
                case "firefox":
                    WebDriverManager.firefoxdriver().setup();
                    FirefoxOptions firefoxOptions = new FirefoxOptions();
                    firefoxOptions.addArguments("-headless");
                    driver = new FirefoxDriver(firefoxOptions);
                    break;
                case "edge":
                    WebDriverManager.edgedriver().setup();
                    EdgeOptions edgeOptions = new EdgeOptions();
                    edgeOptions.addArguments("--remote-allow-origins=*");
                    edgeOptions.addArguments("--headless");
                    edgeOptions.addArguments("--window-size=1920,1080");
                    driver = new EdgeDriver(edgeOptions);
                    break;
                case "chrome":
                default:
                    WebDriverManager.chromedriver().setup();
                    ChromeOptions chromeOptions = new ChromeOptions();
                    chromeOptions.addArguments("--remote-allow-origins=*");
                    chromeOptions.addArguments("--headless=new");
                    chromeOptions.addArguments("--window-size=1920,1080");
                    driver = new ChromeDriver(chromeOptions);
                    break;
            }
            driver.manage().timeouts().implicitlyWait(Duration.ofSeconds(10));
            driverThreadLocal.set(driver);
        }
        return driverThreadLocal.get();
    }
    /**
     * Retrieves the thread-safe WebDriver instance for the current thread.
     * 
     * @return The WebDriver instance, or null if not initialized
     */
    public static WebDriver getDriver() {
        return driverThreadLocal.get();
    }
    /**
     * Quits the WebDriver instance for the current thread and removes it from ThreadLocal.
     */
    public static void quitDriver() {
        if (driverThreadLocal.get() != null) {
            System.out.println("[DriverFactory] Quitting WebDriver session for thread: " + Thread.currentThread().getId());
            try {
                driverThreadLocal.get().quit();
            } catch (Exception e) {
                System.err.println("[DriverFactory] Error encountered while quitting driver: " + e.getMessage());
            } finally {
                driverThreadLocal.remove();
            }
        }
    }
}
```

### 2. Validation Test Suite (`DriverFactoryTest.java`)
We can test this factory utilizing TestNG configuration annotations. We initialize the driver in `@BeforeMethod` and verify it in the `@Test` methods.

```java
package com.mycodeyatra.tests;
import com.mycodeyatra.driver.DriverFactory;
import org.openqa.selenium.WebDriver;
import org.testng.Assert;
import org.testng.annotations.AfterMethod;
import org.testng.annotations.BeforeMethod;
import org.testng.annotations.Optional;
import org.testng.annotations.Parameters;
import org.testng.annotations.Test;
public class DriverFactoryTest {
    @BeforeMethod
    @Parameters("browser")
    public void setUp(@Optional("chrome") String browser) {
        DriverFactory.initDriver(browser);
    }
    @Test
    public void testDriverFactoryChrome() {
        WebDriver driver = DriverFactory.getDriver();
        Assert.assertNotNull(driver, "Driver instance should not be null!");
        driver.get("https://practice.mycodeyatra.com/#/login");
        String title = driver.getTitle();
        System.out.println("[DriverFactoryTest] Current page title: " + title);
        Assert.assertTrue(title.contains("MyCodeYatra"), "Title should contain MyCodeYatra!");
    }
    @Test
    public void testThreadSafety() {
        long currentThreadId = Thread.currentThread().getId();
        WebDriver driver1 = DriverFactory.getDriver();
        System.out.println("[DriverFactoryTest] Thread ID: " + currentThreadId + " - WebDriver: " + driver1);
        Assert.assertNotNull(driver1, "Driver on current thread should be active.");
    }
    @AfterMethod
    public void tearDown() {
        DriverFactory.quitDriver();
    }
}
```

---

## 🚀 Benefits of Driver Factory Pattern

1. **Thread Isolation**: The use of Java's `ThreadLocal` ensures that multiple browser sessions can run simultaneously on different threads without conflicts.
2. **Centralized Configuration**: All driver creation details (headless arguments, timeouts, capabilities) are encapsulated in a single class.
3. **Prevention of Memory Leaks**: Removing driver threads using `ThreadLocal.remove()` in `quitDriver()` avoids garbage collection leaks during heavy pipeline runs.
