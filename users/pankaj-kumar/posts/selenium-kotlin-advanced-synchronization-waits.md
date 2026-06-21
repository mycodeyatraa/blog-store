---
title: "Advanced Synchronization using WebDriverWait and Kotlin Extensions"
date: "09-Feb-2025"
description: "Eliminate flaky tests by replacing Thread.sleep() with robust WebDriverWait strategies, enhanced by Kotlin Extension functions for maximum readability."
categories: ["Selenium Kotlin", "Test Automation", "Waits"]
tags: ["Selenium", "Kotlin", "Kotest", "WebDriverWait", "Synchronization"]
author: "Pankaj Kumar"
lastUpdated: "09-Feb-2025"
---

Welcome back to the **Selenium Kotlin Mastery** series!

The number one reason UI automation tests fail is **Synchronization**. The test runs faster than the browser can render the DOM. The traditional "hack" is to use `Thread.sleep(5000)`, which completely freezes the main thread, making your test suites painfully slow.

In Kotlin, while you *could* use Coroutines (`delay(5000)`) which are non-blocking to the thread, hardcoded waits are still a bad practice. The correct approach is to use Selenium's `WebDriverWait` (Explicit Waits). 

Today, we will combine `WebDriverWait` with **Kotlin Extension Functions** to create an elegant, robust waiting mechanism.

---

## 1. Creating the Wait Extensions

Let's expand our `WebDriverExtensions.kt` file. We want to be able to say `driver.waitForElementVisible(By.id(...))` instead of typing out the verbose `new WebDriverWait(...)` syntax every single time.

Add the following to `src/main/kotlin/com/mycodeyatra/extensions/WebDriverExtensions.kt`:
```kotlin
import org.openqa.selenium.By
import org.openqa.selenium.WebDriver
import org.openqa.selenium.WebElement
import org.openqa.selenium.support.ui.ExpectedConditions
import org.openqa.selenium.support.ui.WebDriverWait
import java.time.Duration
// Standard Explicit Wait Extension
fun WebDriver.waitForElementVisible(locator: By, timeoutInSeconds: Long = 10): WebElement {
    val wait = WebDriverWait(this, Duration.ofSeconds(timeoutInSeconds))
    return wait.until(ExpectedConditions.visibilityOfElementLocated(locator))
}
fun WebDriver.waitForElementClickable(locator: By, timeoutInSeconds: Long = 10): WebElement {
    val wait = WebDriverWait(this, Duration.ofSeconds(timeoutInSeconds))
    return wait.until(ExpectedConditions.elementToBeClickable(locator))
}
```

### Why this is brilliant:
1. **Default Arguments:** Notice `timeoutInSeconds: Long = 10`. Kotlin allows default parameters! If you just call `driver.waitForElementVisible(By.id("foo"))`, it defaults to 10 seconds. If you need a longer wait for a specific heavy element, you can explicitly pass `timeoutInSeconds = 30`. No method overloading required!

---

## 2. Implementing the Waits in Tests

Let's see this in action. Create `Blog4_SynchronizationTest.kt` in your tests folder:
```kotlin
package com.mycodeyatra.tests
import com.mycodeyatra.extensions.waitForElementClickable
import com.mycodeyatra.extensions.waitForElementVisible
import io.kotest.core.spec.style.StringSpec
import io.kotest.matchers.shouldBe
import org.openqa.selenium.By
import org.openqa.selenium.WebDriver
import org.openqa.selenium.chrome.ChromeDriver
class Blog4_SynchronizationTest : StringSpec({
    lateinit var driver: WebDriver
    beforeTest {
        driver = ChromeDriver()
        driver.manage().window().maximize()
    }
    afterTest {
        driver.quit()
    }
    "Should wait for dynamic elements to load" {
        driver.get("https://mycodeyatra.com/practice/dynamic-loading")
        // 1. Click a button that triggers an AJAX request
        val startButton = driver.findElement(By.id("start-btn"))
        startButton.click()
        // 2. Wait up to 15 seconds for the loader to DISAPPEAR (Custom Wait logic can be added)
        // 3. Wait for the final text to be visible, using default 10s timeout
        val loadedTextElement = driver.waitForElementVisible(By.id("finish-text"))
        // 4. Assert
        loadedTextElement.text shouldBe "Hello World!"
        // 5. Wait for a button to be clickable before clicking it (overriding default to 5s)
        val confirmBtn = driver.waitForElementClickable(By.id("confirm-btn"), timeoutInSeconds = 5)
        confirmBtn.click()
    }
})
```

---

## 3. Implicit vs Explicit Waits

Remember: **Never mix Implicit and Explicit Waits**. 
If you set `driver.manage().timeouts().implicitlyWait(Duration.ofSeconds(10))` globally, and then use `WebDriverWait`, Selenium will experience unpredictable timeout issues where the waits multiply or override each other.

By utilizing our custom Kotlin Extension functions, you keep your framework 100% Explicit, ensuring the test waits *only* when necessary, and proceeds the exact millisecond the condition is met.

---

---

## Expected Output

When executing this test suite via Kotest, you will see the following output in your IntelliJ/Maven console confirming that the tests passed successfully:
```text
[INFO] Running com.mycodeyatra.tests.Blog4_SynchronizationTest
Blog4_SynchronizationTest
[PASS] Should wait for dynamic elements to load
1 tests completed, 1 successes, 0 failures, 0 ignored.
```

## Conclusion

With Kotlin's extension functions and default parameter values, implementing Explicit Waits is no longer a chore. Your tests will now execute as fast as the application allows, without arbitrary `Thread.sleep()` commands causing artificial slowdowns.

In our next blog, we will tackle **Handling Alerts and Javascript Execution**, exploring how Kotlin interoperates seamlessly with the browser's native JS engine!

Happy Automating!
