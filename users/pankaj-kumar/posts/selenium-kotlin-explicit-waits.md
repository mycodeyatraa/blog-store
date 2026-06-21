---
title: "Mastering Explicit Waits using Kotlin Extension Functions"
date: "20-Feb-2025"
description: "Ditch the flaky Thread.sleep() and Implicit Waits! Learn how to build highly readable Explicit and Fluent Waits using powerful Kotlin Extension Functions."
categories: ["Selenium Kotlin", "Test Automation", "Waits"]
tags: ["Selenium", "Kotlin", "Kotest", "Waits", "WebDriverWait", "Explicit Wait"]
author: "Pankaj Kumar"
lastUpdated: "20-Feb-2025"
---

Welcome back to the **Selenium Kotlin Mastery Series**!

In web automation, synchronization is the number one cause of flaky tests. If you try to click a button before the JavaScript has finished rendering it on the screen, your test will instantly crash with an `ElementNotInteractableException`.

While beginners often resort to `Thread.sleep(5000)` (which is a massive anti-pattern) or `driver.manage().timeouts().implicitlyWait()`, true enterprise frameworks rely heavily on **Explicit Waits**. 

Today, we will learn how to wrap Selenium's `WebDriverWait` inside incredibly clean **Kotlin Extension Functions**.

---

## 1. Why Kotlin Extension Functions?

In Java, invoking an explicit wait looks something like this:

```java
WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
WebElement element = wait.until(ExpectedConditions.visibilityOfElementLocated(By.id("username")));
```

Writing this everywhere in your framework creates terrible boilerplate code. With Kotlin, we can attach new methods directly to the `WebDriver` class itself using **Extension Functions**!

Let's create a new file `WebDriverExtensions.kt` in `src/main/kotlin/com/mycodeyatra/utils/`.

```kotlin
package com.mycodeyatra.utils
import org.openqa.selenium.By
import org.openqa.selenium.WebDriver
import org.openqa.selenium.WebElement
import org.openqa.selenium.support.ui.ExpectedConditions
import org.openqa.selenium.support.ui.WebDriverWait
import java.time.Duration
// 1. Extension function for waiting until an element is VISIBLE
fun WebDriver.waitForElementVisible(locator: By, timeoutSeconds: Long = 10): WebElement {
    println("Waiting up to $timeoutSeconds seconds for element: $locator")
    val wait = WebDriverWait(this, Duration.ofSeconds(timeoutSeconds))
    return wait.until(ExpectedConditions.visibilityOfElementLocated(locator))
}
// 2. Extension function for waiting until an element is CLICKABLE
fun WebDriver.waitForElementClickable(locator: By, timeoutSeconds: Long = 10): WebElement {
    println("Waiting up to $timeoutSeconds seconds for element to be clickable: $locator")
    val wait = WebDriverWait(this, Duration.ofSeconds(timeoutSeconds))
    return wait.until(ExpectedConditions.elementToBeClickable(locator))
}
```

By defining these extensions, `driver.waitForElementVisible()` is now natively available on any `WebDriver` instance throughout your entire project!

---

## 2. Using Extension Waits in Tests

Let's test our new extensions in `Blog15_WaitsTest.kt`. 

```kotlin
package com.mycodeyatra.tests
import com.mycodeyatra.utils.waitForElementClickable
import com.mycodeyatra.utils.waitForElementVisible
import io.kotest.core.spec.style.StringSpec
import org.openqa.selenium.By
import org.openqa.selenium.WebDriver
import org.openqa.selenium.chrome.ChromeDriver
import org.openqa.selenium.chrome.ChromeOptions
class Blog15_WaitsTest : StringSpec({
    lateinit var driver: WebDriver
    beforeTest {
        val options = ChromeOptions().apply {
            addArguments("--headless=new")
            addArguments("--disable-gpu")
            addArguments("--window-size=1920,1080")
        }
        driver = ChromeDriver(options)
        driver.manage().window().maximize()
    }
    afterTest {
        driver.quit()
    }
    "Should use Kotlin Extension Functions for Explicit Waits" {
        println("[INFO] Testing Explicit Waits via Extension Functions")
        val dynamicLocator = By.id("dynamic_loader")
        
        // Wait up to 15 seconds for the loader to appear
        try {
            val element = driver.waitForElementVisible(dynamicLocator, 15)
            println("Element is visible!")
        } catch (e: Exception) {
            println("[ERROR] Element never appeared within 15 seconds.")
        }
    }
})
```

Because we set a default value `timeoutSeconds: Long = 10` in our extension function signature, we can optionally omit the second parameter entirely: `driver.waitForElementVisible(dynamicLocator)`. 

---

## Expected Output

When running a test against a slow network request, the logs will cleanly output our custom wait message before executing the condition check!

```text
[INFO] Testing Explicit Waits via Extension Functions
Waiting up to 15 seconds for element: By.id: dynamic_loader
[ERROR] Element never appeared within 15 seconds.
```

## Conclusion

Explicit Waits are non-negotiable in modern automation frameworks. By combining `WebDriverWait` with Kotlin Extension Functions, we eliminate verbose boilerplate and create a domain-specific language (DSL) that is beautifully readable!

In our next blog, we will tackle one of the trickiest topics in web automation: **Managing Iframes and Multiple Windows** seamlessly in Kotlin!

Happy Automating!
