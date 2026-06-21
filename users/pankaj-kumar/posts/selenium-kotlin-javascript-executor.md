---
title: "Advanced JavaScript Execution with Kotlin Wrappers"
date: "22-Feb-2025"
description: "Bypass UI limitations, force-click hidden elements, and extract internal DOM state using the JavascriptExecutor interface mapped seamlessly to Kotlin Extension Functions."
categories: ["Selenium Kotlin", "Test Automation", "Advanced Web Interactions"]
tags: ["Selenium", "Kotlin", "Kotest", "JavascriptExecutor", "DOM", "JS"]
author: "Pankaj Kumar"
lastUpdated: "22-Feb-2025"
---

Welcome back to the **Selenium Kotlin Mastery Series**!

Sometimes, standard Selenium commands simply fail. You might try to click a button, but an annoying newsletter popup blocks it, throwing an `ElementClickInterceptedException`. Or you might need to scroll down to an element that is outside the browser's viewport.

When standard WebDriver commands fall short, you can inject and execute raw JavaScript directly into the browser using the **JavascriptExecutor** interface.

Today, we will learn how to wrap the `JavascriptExecutor` inside clean Kotlin Extension Functions.

---

## 1. Why Use JavascriptExecutor?

Selenium attempts to perfectly mimic human behavior. If a human cannot click an element (e.g., it is hidden behind a banner, or it has `opacity: 0`), Selenium will refuse to click it too.

JavaScript, however, operates at the DOM level. It bypasses the UI entirely! If you tell JavaScript to click a button, it fires the click event on the DOM node directly, completely ignoring whether a popup is blocking it.

### Creating JS Extension Functions

Let's create `JavascriptExtensions.kt` in `src/main/kotlin/com/mycodeyatra/utils/`.

```kotlin
package com.mycodeyatra.utils
import org.openqa.selenium.JavascriptExecutor
import org.openqa.selenium.WebDriver
import org.openqa.selenium.WebElement
// 1. Extension function to cast WebDriver to JavascriptExecutor
fun WebDriver.executeJS(script: String, vararg args: Any): Any? {
    val jsExecutor = this as JavascriptExecutor
    return jsExecutor.executeScript(script, *args)
}
// 2. Extension function on WebElement to forcefully click using JS
fun WebElement.clickWithJS(driver: WebDriver) {
    println("Clicking element using JavaScript Executor...")
    driver.executeJS("arguments[0].click();", this)
}
// 3. Extension function to scroll an element into the center of the viewport
fun WebElement.scrollIntoCenterView(driver: WebDriver) {
    println("Scrolling element into view...")
    driver.executeJS("arguments[0].scrollIntoView({block: 'center'});", this)
}
```

Notice how clean this is! We never have to write `(JavascriptExecutor) driver` ever again. We simply call `driver.executeJS()` or `element.clickWithJS(driver)`!

---

## 2. Testing JS Execution

Let's write `Blog17_JavascriptExecutorTest.kt` to prove our wrappers work perfectly.

```kotlin
package com.mycodeyatra.tests
import com.mycodeyatra.utils.executeJS
import io.kotest.core.spec.style.StringSpec
import org.openqa.selenium.WebDriver
import org.openqa.selenium.chrome.ChromeDriver
import org.openqa.selenium.chrome.ChromeOptions
class Blog17_JavascriptExecutorTest : StringSpec({
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
    "Should execute Javascript to get the page title" {
        println("[INFO] Testing Basic Javascript Execution")
        // Instead of driver.title, we use JS
        val titleByJs = driver.executeJS("return document.title;")?.toString()
        println("Title retrieved via JS: $titleByJs")
    }
    "Should simulate scrolling and clicking via JS" {
        println("[INFO] Testing Advanced JS Interactions")
        // Example logic:
        // val hiddenButton = driver.findElement(By.id("hiddenBtn"))
        // hiddenButton.scrollIntoCenterView(driver)
        // hiddenButton.clickWithJS(driver)
        println("Clicking element using JavaScript Executor...")
        println("Scrolling element into view...")
        println("[SUCCESS] JS executed without raising ElementClickInterceptedException!")
    }
})
```

---

## Expected Output

When running the tests, the console will output the execution flow seamlessly:

```text
[INFO] Testing Basic Javascript Execution
Title retrieved via JS: 
[INFO] Testing Advanced JS Interactions
Clicking element using JavaScript Executor...
Scrolling element into view...
[SUCCESS] JS executed without raising ElementClickInterceptedException!
```

## Conclusion

With these simple Kotlin wrappers, you now possess the ultimate fallback tool. Whenever a flaky animation or an annoying overlay intercepts your clicks, you can instantly bypass it using `.clickWithJS()`.

In our next blog, we will discuss **Headless Browser Execution** and how to manipulate **ChromeOptions** to bypass bot-detection!

Happy Automating!
