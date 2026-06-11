---
title: Capturing Screenshots and Failure Evidence in Selenium
date: 10-Jul-2024
lastUpdated: 11-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [selenium, java, screenshots, logging, debugging]
category: UI Automation
categories: [UI Automation, Selenium, Java]
excerpt: >-
  Learn how to collect screenshots and automated failure evidence in your test suites using Selenium WebDriver in Java, featuring a complete step-by-step TestNG automation guide.
readTime: 7 min read
---

# Capturing Screenshots and Failure Evidence in Selenium

> 📅 **Last Updated:** 11-Jun-2026

When automated test suites run in headless environments like CI/CD containers, a failure log is rarely enough to diagnose a bug. Having a visual snapshot of the application state at the exact moment of failure is essential.

In this guide, we will cover how to capture full-page screenshots, target specific HTML elements, and design failure evidence capture pipelines using Selenium WebDriver and TestNG.

---

## 📸 Failure Evidence Capture Flow

When an assertion fails, the test framework must handle the error, halt normal execution, capture the visual state of the DOM, persist it to disk, and attach the log path to reports before cleaning up the session.

![diagram_1](https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/posts/capturing-screenshots-failure-evidence-selenium/images/diagram_1.png)

---

## 🛠️ Step-by-Step Code Walkthrough

In Selenium 4, screenshots are divided into two main execution paths:
1. **Full-page snapshotting**: Achieved by casting the `WebDriver` instance to `TakesScreenshot`.
2. **Element-specific snapshotting**: Invoked directly on any active `WebElement` instance.

Here is the complete Java test showing both approaches:

```java
package com.mycodeyatra.tests;
import io.github.bonigarcia.wdm.WebDriverManager;
import org.openqa.selenium.By;
import org.openqa.selenium.OutputType;
import org.openqa.selenium.TakesScreenshot;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.chrome.ChromeOptions;
import org.openqa.selenium.io.FileHandler;
import org.testng.Assert;
import org.testng.annotations.AfterMethod;
import org.testng.annotations.BeforeMethod;
import org.testng.annotations.Test;
import java.io.File;
import java.io.IOException;
import java.time.Duration;
public class ScreenshotTest {
    private WebDriver driver;
    @BeforeMethod
    public void setUp() {
        WebDriverManager.chromedriver().setup();
        ChromeOptions options = new ChromeOptions();
        options.addArguments("--remote-allow-origins=*");
        options.addArguments("--headless=new"); // Capture cleanly in headless CI runners
        driver = new ChromeDriver(options);
        driver.manage().window().maximize();
        driver.manage().timeouts().implicitlyWait(Duration.ofSeconds(10));
    }
    @Test
    public void testScreenshotCapture() throws IOException {
        System.out.println("Navigating to: https://practice.mycodeyatra.com/");
        driver.get("https://practice.mycodeyatra.com/");
        // Create target screenshot folder inside target directory
        File screenshotDir = new File("target/screenshots");
        if (!screenshotDir.exists()) {
            screenshotDir.mkdirs();
        }
        // 1. Capture Full Page Screenshot
        System.out.println("Capturing full page screenshot...");
        File srcFile = ((TakesScreenshot) driver).getScreenshotAs(OutputType.FILE);
        File destFile = new File("target/screenshots/full_page_screenshot.png");
        FileHandler.copy(srcFile, destFile);
        System.out.println("Full page screenshot saved to: " + destFile.getAbsolutePath());
        Assert.assertTrue(destFile.exists(), "Full page screenshot file was not created.");
        // 2. Capture Element Specific Screenshot (Selenium 4 Feature)
        System.out.println("Locating sandbox header element...");
        WebElement headerElement = driver.findElement(By.xpath("//h1"));
        System.out.println("Capturing specific element screenshot...");
        File elementSrcFile = headerElement.getScreenshotAs(OutputType.FILE);
        File elementDestFile = new File("target/screenshots/element_header_screenshot.png");
        FileHandler.copy(elementSrcFile, elementDestFile);
        System.out.println("Element screenshot saved to: " + elementDestFile.getAbsolutePath());
        Assert.assertTrue(elementDestFile.exists(), "Element screenshot file was not created.");
        System.out.println("Screenshot capture tests executed successfully!");
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

Here are the console logs from compiling and running the screenshot suite:

```bash
[INFO] Running com.mycodeyatra.tests.ScreenshotTest
Navigating to: https://practice.mycodeyatra.com/
Capturing full page screenshot...
Full page screenshot saved to: D:\MyCodeYatra\AILearning2026\Repository\mcyt-sel-java\target\screenshots\full_page_screenshot.png
Locating sandbox header element...
Capturing specific element screenshot...
Element screenshot saved to: D:\MyCodeYatra\AILearning2026\Repository\mcyt-sel-java\target\screenshots\element_header_screenshot.png
Screenshot capture tests executed successfully!
Quitting driver session...
Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 4.887 sec
```

---

## 📊 Comparison of Evidence Strategies

Capturing evidence requires balancing coverage against disk storage limits.

| Capture Strategy | Coverage Scope | Storage Footprint | Primary Use Case |
| :--- | :--- | :--- | :--- |
| **Full Page Screenshot** | Complete visible browser viewport | Medium (150KB - 500KB) | Standard failure trace evidence |
| **Element Screenshot** | Targeted DOM node boundaries only | Low (20KB - 80KB) | Component state auditing |
| **Console Logs** | JavaScript warnings/errors list | Very Low (< 10KB) | Identifying code script crashes |
| **Video Recording** | Interactive user session video | High (1MB - 10MB) | Complex asynchronous debugs |

---

## ⚠️ Common Pitfalls

* **Relative Directory Discrepancies**: Always build output directories dynamically (using path variables relative to the project directory) instead of hardcoding root Windows folders.
* **Storage Exhaustion**: Capturing PNGs for every passing test in a suite of 1000+ tests can consume gigabytes of storage on CI servers. Filter the capture scripts to execute exclusively on failure bounds.
* **Invisible Elements in Headless Mode**: Headless browser windows defaults to `800x600` resolution. If elements are located outside this viewport, standard screenshot captures might cut off. Configure options via `options.addArguments("--window-size=1920,1080")` to prevent truncation.

---

## 🙋 Frequently Asked Questions

> **Q: How can we automate screenshot capturing on test failure automatically without editing every test?**
>
> **A:** Implement TestNG's `ITestListener` interface. Override the `onTestFailure(ITestResult result)` method to query the driver instance and write the screenshot, then register the listener in your `testng.xml` file.

> **Q: Can we capture screenshots of closed frames/iFrames?**
>
> **A:** No. You must switch context into the target frame (using `driver.switchTo().frame()`) before attempting to locate the element or capture its element-specific screenshot.
