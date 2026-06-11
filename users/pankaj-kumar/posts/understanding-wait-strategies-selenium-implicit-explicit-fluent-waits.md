---
title: Understanding Wait Strategies in Selenium: Implicit, Explicit, and Fluent Waits
date: 11-Jul-2024
lastUpdated: 11-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [selenium, java, waits, implicit-wait, explicit-wait, fluent-wait]
category: UI Automation
categories: [UI Automation, Selenium, Java]
excerpt: >-
  Master synchronization in Selenium WebDriver. Learn the structural and runtime differences between Implicit, Explicit, and Fluent Waits, and how to automate dynamic elements in Java.
readTime: 7 min read
---

# Understanding Wait Strategies in Selenium: Implicit, Explicit, and Fluent Waits

> 📅 **Last Updated:** 11-Jun-2026

Modern web interfaces are fundamentally dynamic. Content loads asynchronously via AJAX requests, React hooks, or background API fetch streams. If an automation script attempts to click an element that hasn't finished rendering, it throws the notorious `NoSuchElementException` or `ElementNotInteractableException`. 

To write robust, non-flaky test suites, you must understand how to synchronize your tests with the application state using Selenium's synchronization strategies.

---

## 🚦 Wait Architectures: Explicit vs. Fluent Waits

The difference between Explicit Wait and Fluent Wait is a matter of customization:

![diagram_1](https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/posts/understanding-wait-strategies-selenium-implicit-explicit-fluent-waits/images/diagram_1.png)

---

## 🛠️ Step-by-Step Code Walkthrough

Here is the complete TestNG implementation demonstrating:
1. **Implicit Wait**: Configured globally to handle general page transitions.
2. **Explicit Wait (`WebDriverWait`)**: Used for waiting on standard 3-second delayed elements.
3. **Fluent Wait**: Used for custom polling checks as a progress bar increases from `0%` to `100%`.

```java
package com.mycodeyatra.tests;
import io.github.bonigarcia.wdm.WebDriverManager;
import org.openqa.selenium.By;
import org.openqa.selenium.NoSuchElementException;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.chrome.ChromeOptions;
import org.openqa.selenium.support.ui.ExpectedConditions;
import org.openqa.selenium.support.ui.FluentWait;
import org.openqa.selenium.support.ui.Wait;
import org.openqa.selenium.support.ui.WebDriverWait;
import org.testng.Assert;
import org.testng.annotations.AfterMethod;
import org.testng.annotations.BeforeMethod;
import org.testng.annotations.Test;
import java.time.Duration;
public class WaitTest {
    private WebDriver driver;
    private WebDriverWait explicitWait;
    @BeforeMethod
    public void setUp() {
        WebDriverManager.chromedriver().setup();
        ChromeOptions options = new ChromeOptions();
        options.addArguments("--remote-allow-origins=*");
        options.addArguments("--headless=new");
        driver = new ChromeDriver(options);
        driver.manage().window().maximize();
        // 1. Implicit Wait Configuration (10 seconds fallback)
        driver.manage().timeouts().implicitlyWait(Duration.ofSeconds(10));
        // 2. Explicit Wait Configuration (5 seconds timeout)
        explicitWait = new WebDriverWait(driver, Duration.ofSeconds(5));
    }
    @Test
    public void testWaitStrategies() {
        System.out.println("Navigating to: https://practice.mycodeyatra.com/");
        driver.get("https://practice.mycodeyatra.com/");
        System.out.println("Opening Sandbox Arena...");
        WebElement sandboxLink = explicitWait.until(ExpectedConditions.elementToBeClickable(By.xpath("//span[contains(text(),'Sandbox Arena')]")));
        sandboxLink.click();
        System.out.println("Opening Dynamic Content & Loading Tile...");
        WebElement dynamicContentTile = explicitWait.until(ExpectedConditions.elementToBeClickable(By.xpath("//h3[text()='Dynamic Content']")));
        dynamicContentTile.click();
        // --- PHASE 1: EXPLICIT WAIT DEMO ---
        System.out.println("--- Starting Explicit Wait Demo ---");
        WebElement startLoadingBtn = driver.findElement(By.xpath("//button[@data-testid='start-loading-btn']"));
        startLoadingBtn.click();
        // Wait for the delayed content element to appear (expected delay 3s)
        System.out.println("Waiting for delayed content to appear (max 5s)...");
        WebElement delayedContent = explicitWait.until(
            ExpectedConditions.visibilityOfElementLocated(By.xpath("//div[@data-testid='delayed-content']"))
        );
        String contentText = delayedContent.getText();
        System.out.println("Delayed Content loaded: " + contentText);
        Assert.assertEquals(contentText, "Content loaded after 3 seconds!", "Failed to fetch delayed content.");
        // --- PHASE 2: FLUENT WAIT DEMO ---
        System.out.println("--- Starting Fluent Wait Demo ---");
        WebElement startProgressBtn = driver.findElement(By.xpath("//button[@data-testid='start-progress-btn']"));
        startProgressBtn.click();
        // Configure Fluent Wait: 10s timeout, 500ms polling, ignore NoSuchElementException
        System.out.println("Configuring Fluent Wait (timeout 10s, polling 500ms)...");
        Wait<WebDriver> fluentWait = new FluentWait<>(driver)
                .withTimeout(Duration.ofSeconds(10))
                .pollingEvery(Duration.ofMillis(500))
                .ignoring(NoSuchElementException.class);
        // Wait for progress text to reach 100%
        WebElement progressText = fluentWait.until(d -> {
            WebElement element = d.findElement(By.xpath("//strong[@data-testid='progress-text']"));
            String progressVal = element.getText();
            System.out.println("Fluent polling progress: " + progressVal);
            if ("100%".equals(progressVal)) {
                return element;
            }
            return null;
        });
        Assert.assertNotNull(progressText, "Progress failed to reach 100% within timeout.");
        // Click completed progress button
        WebElement progressCompleteBtn = driver.findElement(By.xpath("//button[@data-testid='progress-complete-btn']"));
        Assert.assertTrue(progressCompleteBtn.isEnabled(), "Complete button is still disabled.");
        progressCompleteBtn.click();
        System.out.println("Progress button clicked successfully!");
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

Here are the console outputs of the test execution, showcasing the Fluent Wait polling cycle:

```bash
[INFO] Running com.mycodeyatra.tests.WaitTest
Navigating to: https://practice.mycodeyatra.com/
Opening Sandbox Arena...
Opening Dynamic Content & Loading Tile...
--- Starting Explicit Wait Demo ---
Waiting for delayed content to appear (max 5s)...
Delayed Content loaded: Content loaded after 3 seconds!
--- Starting Fluent Wait Demo ---
Configuring Fluent Wait (timeout 10s, polling 500ms)...
Fluent polling progress: 0%
Fluent polling progress: 0%
Fluent polling progress: 10%
Fluent polling progress: 20%
Fluent polling progress: 30%
Fluent polling progress: 40%
Fluent polling progress: 50%
Fluent polling progress: 60%
Fluent polling progress: 70%
Fluent polling progress: 80%
Fluent polling progress: 90%
Fluent polling progress: 100%
Progress button clicked successfully!
Quitting driver session...
Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 13.563 sec
```

---

## 📊 Summary of Wait Strategies

Understanding when to apply each wait ensures optimal performance in your test runtimes.

| Metric | Implicit Wait | Explicit Wait | Fluent Wait |
| :--- | :--- | :--- | :--- |
| **Scope** | Global (all elements) | Local (targeted element) | Local (targeted element) |
| **Timeout Configuration** | Set once per session | Set per action | Set per action |
| **Polling Interval** | Constant (internal driver) | Fixed (500ms in Java) | Customizable (e.g. 100ms, 1s) |
| **Custom Conditions** | None | Limited (ExpectedConditions) | Unlimited (Predicate functions) |
| **Exception Handling** | None | Throws TimeoutException | Customizable (ignores specified classes) |

---

## ⚠️ Common Pitfalls

* **Mixing Implicit and Explicit Waits**: Mixing implicit and explicit waits is highly discouraged. Doing so causes the driver communication loop to enter unexpected sleep periods, multiplying wait times or resulting in unexpected `TimeoutExceptions`.
* **Setting High Implicit Timeouts**: Setting a global implicit wait of 30 seconds sounds safe, but it makes negative validations (asserting that an element does *not* exist) incredibly slow, as Selenium is forced to wait the entire 30 seconds before proceeding.
* **Ignoring the Wrong Exceptions**: By default, Fluent Wait ignores no exceptions. If you do not specify `.ignoring(NoSuchElementException.class)`, the first polling cycle that fails to locate the element will crash the test immediately.

---

## 🙋 Frequently Asked Questions

> **Q: What is the difference between Fluent Wait and Explicit Wait?**
>
> **A:** Under the hood, Selenium's `WebDriverWait` class is actually a specialized subclass of `FluentWait`. However, `FluentWait` exposes custom configurations like polling intervals and ignored exception lists, whereas `WebDriverWait` relies on default values (500ms polling).

> **Q: Does Thread.sleep() count as a Selenium wait strategy?**
>
> **A:** No. `Thread.sleep()` is a Java thread pause. It halts execution for the hardcoded duration regardless of whether the element renders in 10ms or 5s. This is a severe anti-pattern that slows down test suites and causes flakiness. Always use dynamic waits.
