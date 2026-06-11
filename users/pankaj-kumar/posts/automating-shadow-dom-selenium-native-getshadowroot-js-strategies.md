---
title: Automating Shadow DOM in Selenium WebDriver: Native getShadowRoot and JS Strategies
date: 18-Jul-2024
lastUpdated: 11-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [selenium, java, shadow-dom, getshadowroot, web-components, javascript-executor]
category: UI Automation
categories: [UI Automation, Selenium, Java]
excerpt: >-
  A complete guide to automating Shadow DOM elements in Selenium WebDriver. Learn how to use Selenium 4's native getShadowRoot API and JavaScript Executor to bypass DOM boundaries in Java.
readTime: 7 min read
---

# Automating Shadow DOM in Selenium WebDriver: Native getShadowRoot and JS Strategies

> 📅 **Last Updated:** 11-Jun-2026

Modern web frameworks (like Angular, React, and Salesforce LWC) make extensive use of Web Components. At the heart of these web components lies the **Shadow DOM**, which provides document encapsulation. It keeps the markup structure, style, and behavior of a component hidden and separated from other code on the page.

However, this encapsulation creates a boundary that standard Selenium locator strategies (especially XPath) cannot cross. If you try to find an element inside a shadow root using a standard XPath selector, Selenium throws a `NoSuchElementException`. In this tutorial, we will learn how to bypass this boundary in Java.

---

## 🧭 Shadow DOM Interaction Flowchart

The diagram below outlines how the driver traverses the document boundary by targeting the Shadow Host, retrieving its Shadow Root context, and locating the inner element:

![diagram_1](https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/posts/automating-shadow-dom-selenium-native-getshadowroot-js-strategies/images/diagram_1.png)

---

## 🛠️ Step-by-Step Code Walkthrough

To automate Shadow DOM elements, we use the following Selenium 4 API:
1. `WebElement shadowHost = driver.findElement(By.id("shadow-host"));` (Locates the element hosting the shadow boundary).
2. `SearchContext shadowRoot = shadowHost.getShadowRoot();` (Returns a `SearchContext` representing the shadow root).
3. `WebElement shadowInput = shadowRoot.findElement(By.cssSelector("#shadow-input"));` (Performs a find query relative to the shadow root).

> ⚠️ **Important:** Inside a `SearchContext` returned by `getShadowRoot()`, you cannot use XPath queries. You must use CSS Selectors, Tag Names, or IDs.

Here is the complete TestNG implementation showcasing both Selenium 4 native API calls and JavaScript Executor fallbacks:

```java
package com.mycodeyatra.tests;
import io.github.bonigarcia.wdm.WebDriverManager;
import org.openqa.selenium.By;
import org.openqa.selenium.JavascriptExecutor;
import org.openqa.selenium.PageLoadStrategy;
import org.openqa.selenium.SearchContext;
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
public class ShadowDomTest {
    private WebDriver driver;
    private WebDriverWait wait;
    @BeforeMethod
    public void setUp() {
        WebDriverManager.chromedriver().setup();
        ChromeOptions options = new ChromeOptions();
        options.addArguments("--remote-allow-origins=*");
        options.addArguments("--headless=new");
        options.addArguments("--window-size=1920,1080");
        options.setPageLoadStrategy(PageLoadStrategy.NONE);
        driver = new ChromeDriver(options);
        driver.manage().timeouts().implicitlyWait(Duration.ofSeconds(10));
        wait = new WebDriverWait(driver, Duration.ofSeconds(10));
    }
    @Test
    public void testInteractWithShadowDomNative() {
        String practiceUrl = "https://practice.mycodeyatra.com/#/shadow-dom";
        System.out.println("Navigating to Live Shadow DOM URL: " + practiceUrl);
        driver.get(practiceUrl);
        // 1. Locate the shadow host element
        System.out.println("Locating shadow host element...");
        WebElement shadowHost = wait.until(
            ExpectedConditions.presenceOfElementLocated(By.id("shadow-host"))
        );
        // 2. Fetch the SearchContext representing the Shadow Root (Selenium 4 Feature)
        System.out.println("Accessing Shadow Root natively...");
        SearchContext shadowRoot = shadowHost.getShadowRoot();
        // 3. Locate elements inside the Shadow Root using CSS Selector
        WebElement shadowInput = shadowRoot.findElement(By.cssSelector("#shadow-input"));
        WebElement shadowBtn = shadowRoot.findElement(By.cssSelector("#shadow-btn"));
        WebElement shadowMsg = shadowRoot.findElement(By.cssSelector("#shadow-msg"));
        // 4. Perform actions inside the Shadow DOM
        String textToType = "Hello MyCodeYatra Shadow DOM!";
        System.out.println("Typing text inside Shadow DOM input...");
        shadowInput.sendKeys(textToType);
        System.out.println("Clicking submit button inside Shadow DOM...");
        shadowBtn.click();
        // 5. Verify the results
        String actualMsg = shadowMsg.getText();
        System.out.println("Message displayed inside Shadow DOM: " + actualMsg);
        Assert.assertEquals(actualMsg, "You typed: " + textToType, "Shadow DOM validation failed!");
        System.out.println("Native Shadow DOM test completed successfully!");
    }
    @Test
    public void testInteractWithShadowDomJSExecutor() {
        String practiceUrl = "https://practice.mycodeyatra.com/#/shadow-dom";
        System.out.println("Navigating to Live Shadow DOM URL: " + practiceUrl);
        driver.get(practiceUrl);
        // 1. Locate the shadow host element
        WebElement shadowHost = wait.until(
            ExpectedConditions.presenceOfElementLocated(By.id("shadow-host"))
        );
        // 2. Locate input and button elements inside the Shadow Root using JavascriptExecutor
        System.out.println("Accessing Shadow Root elements via JavascriptExecutor...");
        JavascriptExecutor js = (JavascriptExecutor) driver;
        WebElement shadowInput = (WebElement) js.executeScript(
            "return arguments[0].shadowRoot.querySelector('#shadow-input');", shadowHost
        );
        WebElement shadowBtn = (WebElement) js.executeScript(
            "return arguments[0].shadowRoot.querySelector('#shadow-btn');", shadowHost
        );
        // 3. Interact with elements
        String textToType = "JS Executor Shadow Text";
        System.out.println("Typing text via Selenium using the retrieved JS Element...");
        shadowInput.sendKeys(textToType);
        shadowBtn.click();
        // 4. Validate output message
        WebElement shadowMsg = (WebElement) js.executeScript(
            "return arguments[0].shadowRoot.querySelector('#shadow-msg');", shadowHost
        );
        String actualMsg = shadowMsg.getText();
        System.out.println("Message displayed: " + actualMsg);
        Assert.assertEquals(actualMsg, "You typed: " + textToType, "JS Shadow DOM validation failed!");
        System.out.println("JS Executor Shadow DOM test completed successfully!");
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

Below is the console output captured from running the Shadow DOM suite:

```bash
[INFO] Running com.mycodeyatra.tests.ShadowDomTest
Navigating to Live Shadow DOM URL: https://practice.mycodeyatra.com/#/shadow-dom
Accessing Shadow Root elements via JavascriptExecutor...
Typing text via Selenium using the retrieved JS Element...
Message displayed: You typed: JS Executor Shadow Text
JS Executor Shadow DOM test completed successfully!
Quitting driver session...
Navigating to Live Sandbox Site: https://practice.mycodeyatra.com/#/sandbox
Navigating to Shadow DOM page...
Locating shadow host element...
Accessing Shadow Root natively...
Typing text inside Shadow DOM input...
Clicking submit button inside Shadow DOM...
Message displayed inside Shadow DOM: You typed: Hello MyCodeYatra Shadow DOM!
Native Shadow DOM test completed successfully!
Quitting driver session...
Tests run: 2, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 10.038 sec
```

---

## 📊 Native `getShadowRoot()` vs JavaScript Executor

Comparing the two primary automation strategies:

| Selection Criteria | Selenium 4 Native `getShadowRoot()` | JavaScript Executor Fallback |
| :--- | :--- | :--- |
| **API Complexity** | Clean, object-oriented Java code. | Requires writing raw JS query selector strings. |
| **Supported Locators** | `By.cssSelector`, `By.tagName`, `By.id`. | Any CSS selector supported by `document.querySelector`. |
| **Selenium Dependency** | Selenium 4.0.0 or higher. | Compatible with Selenium 3.x and 4.x. |
| **XPath Support** | ❌ Unsupported (Throws exception). | ❌ Unsupported (browser querySelector doesn't support XPath). |

---

## ⚠️ Common Pitfalls

* **Attempting to use XPath inside the Shadow Root**: XPath operates on the document level and cannot parse across shadow boundaries. Attempting to use `shadowRoot.findElement(By.xpath("//input"))` will throw an `InvalidArgumentException`. Always use `By.cssSelector` inside a shadow root.
* **Closed Shadow DOM boundaries**: Shadow DOM has two modes: `open` and `closed`. Open mode allows runtime code (and Selenium) to query `shadowRoot`. Closed mode prevents access, returning `null`. If a component uses a closed shadow DOM, standard automation tools cannot target it natively.
* **Implicit waits inside Shadow Roots**: Selenium's implicit waits do not apply to searches executed on a child `SearchContext`. You should wait for the shadow host to exist using standard `WebDriverWait` before retrieving the shadow root.

---

## 🙋 Frequently Asked Questions

> **Q: What can I do if a web component is inside a closed Shadow DOM?**
>
> **A:** If the shadow DOM is closed, `shadowRoot` is inaccessible. You must either ask developers to change the mode to `open` for testability, or use physical keystroke simulation tools (like the Java `Robot` class or Action API chains) to tab into and interact with the elements.

> **Q: Can I chain multiple Shadow Roots if they are nested?**
>
> **A:** Yes! Locate the first shadow host, retrieve its shadow root, use that context to locate the second shadow host inside the first, and then call `getShadowRoot()` on the nested host to open up the sub-boundary.
