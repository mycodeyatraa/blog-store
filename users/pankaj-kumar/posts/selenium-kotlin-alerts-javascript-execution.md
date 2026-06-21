---
title: "Handling Alerts and Javascript Execution in Kotlin"
date: "10-Feb-2025"
description: "Learn how to interact with browser alerts and inject raw Javascript into the page using Kotlin's smart casting and extension functions."
categories: ["Selenium Kotlin", "Test Automation", "Javascript"]
tags: ["Selenium", "Kotlin", "Kotest", "Alerts", "JavascriptExecutor"]
author: "Pankaj Kumar"
lastUpdated: "10-Feb-2025"
---

Welcome back to the **Selenium Kotlin Mastery** series!

Occasionally, you will encounter UI elements that Selenium cannot interact with natively. This includes **Browser Alerts** (which are outside the DOM) and elements hidden behind complex shadow roots where injecting raw **Javascript** becomes the only reliable solution.

Today, we will learn how to handle both of these scenarios gracefully using idiomatic Kotlin.

---

## 1. Handling Browser Alerts

A JavaScript Alert (`window.alert()`) completely blocks the browser until it is dismissed. Selenium handles this by temporarily switching context.

Let's create `Blog5_AlertsAndJsTest.kt` in `src/test/kotlin/com/mycodeyatra/tests/`:

```kotlin
package com.mycodeyatra.tests
import io.kotest.core.spec.style.StringSpec
import io.kotest.matchers.shouldBe
import org.openqa.selenium.By
import org.openqa.selenium.WebDriver
import org.openqa.selenium.chrome.ChromeDriver
class Blog5_AlertsAndJsTest : StringSpec({
    lateinit var driver: WebDriver
    beforeTest {
        driver = ChromeDriver()
        driver.manage().window().maximize()
    }
    afterTest {
        driver.quit()
    }
    "Should accept a JS Prompt Alert" {
        driver.get("https://mycodeyatra.com/practice/alerts")
        // 1. Trigger the alert
        driver.findElement(By.id("prompt-btn")).click()
        // 2. Switch context to the Alert
        val alert = driver.switchTo().alert()
        // 3. Verify text, type into it, and Accept
        alert.text shouldBe "Please enter your name:"
        alert.sendKeys("Pankaj")
        alert.accept() // or alert.dismiss() to cancel
        // 4. Verify result
        driver.findElement(By.id("result")).text shouldBe "Hello Pankaj"
    }
```

---

## 2. Executing Javascript (The Kotlin Way)

In Java, executing Javascript is famously ugly because you have to cast the `WebDriver` object:
`((JavascriptExecutor) driver).executeScript("window.scrollBy(0, 500)");`

In Kotlin, we can completely hide this ugly cast by writing an **Extension Function**! Add this to your `WebDriverExtensions.kt`:

```kotlin
import org.openqa.selenium.JavascriptExecutor
import org.openqa.selenium.WebDriver
// Inline Javascript execution extension
fun WebDriver.executeJs(script: String, vararg args: Any): Any? {
    return (this as JavascriptExecutor).executeScript(script, *args)
}
```

### Why this is awesome:
- `this as JavascriptExecutor`: Kotlin's `as` keyword handles the type casting cleanly.
- `vararg args: Any`: We can pass any number of arguments to the script. The `*args` syntax is Kotlin's "spread operator", which unpacks the array.
- `Any?`: It can return a String, Boolean, WebElement, or null.

### Using the Extension

Now, let's use it in our test file to scroll the page and click a hidden element:

```kotlin
    "Should execute Javascript to scroll and click" {
        driver.get("https://mycodeyatra.com/practice/scrolling")
        // 1. Scroll down by 500 pixels using our shiny new extension!
        driver.executeJs("window.scrollBy(0, 500)")
        // 2. Click a hidden element by passing it as an argument
        val hiddenButton = driver.findElement(By.id("hidden-submit"))
        // Notice we pass the WebElement as an argument to the JS execution
        driver.executeJs("arguments[0].click();", hiddenButton)
    }
})
```

---

---

## Expected Output

When executing this test suite via Kotest, you will see the following output in your IntelliJ/Maven console confirming that the tests passed successfully:

```text
[INFO] Running com.mycodeyatra.tests.Blog5_AlertsAndJsTest
Blog5_AlertsAndJsTest
  [PASS] Should accept a JS Prompt Alert
  [PASS] Should execute Javascript to scroll and click
2 tests completed, 2 successes, 0 failures, 0 ignored.
```

## Conclusion

By leveraging Kotlin's `as` keyword and extension functions, we transformed the ugly, boilerplate-heavy `JavascriptExecutor` cast into a beautiful, one-line `executeJs()` method that feels like a native WebDriver command.

In the next blog, we will master **Windows and iFrames**, learning how to juggle multiple browser contexts effortlessly!

Happy Automating!
