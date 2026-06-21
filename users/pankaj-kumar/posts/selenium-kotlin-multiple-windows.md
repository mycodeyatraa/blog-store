---
title: "Handling Multiple Browser Tabs and Windows in Selenium Kotlin"
date: "2025-02-28"
description: "Master window handles in Selenium Kotlin to seamlessly switch between multiple browser tabs and windows during automated testing."
tags: ["Selenium", "Kotlin", "Windows", "Tabs", "Automation", "Kotest"]
---

Modern web workflows often involve clicking a link that opens a new browser tab or popup window—think of social logins, payment gateways, or external documentation links.

In Selenium, your WebDriver instance is strictly bound to a single active window at any given time. If a new tab opens, Selenium doesn't automatically follow it. You must explicitly tell the driver to switch its context to the new window.

In this 23rd post of our Selenium Kotlin Mastery Series, we will explore Window Handles and how to juggle multiple browser windows effortlessly.

### Understanding Window Handles

Every window or tab opened in a browser session is assigned a unique alphanumeric string by the browser, known as a **Window Handle**. 

You interact with them using two primary methods:
* `driver.windowHandle`: Returns the ID of the window currently in focus.
* `driver.windowHandles`: Returns a `Set<String>` containing the IDs of *all* currently open windows.

### Navigating Between Windows

Let's look at an automated scenario using [The Internet's Multiple Windows page](https://the-internet.herokuapp.com/windows). We will open a new window, verify its contents, close it, and return to the parent window.

```kotlin
package com.mycodeyatra.tests
import com.mycodeyatra.utils.DriverManager
import com.mycodeyatra.utils.waitForElementVisible
import io.kotest.core.spec.style.StringSpec
import io.kotest.matchers.shouldBe
import org.openqa.selenium.By
import org.openqa.selenium.WebDriver
class Blog23_WindowsTest : StringSpec({
    var driver: WebDriver? = null
    beforeSpec {
        driver = DriverManager.getHeadlessChromeDriver()
        driver?.manage()?.window()?.maximize()
    }
    afterSpec {
        driver?.quit()
    }
    "Switch between multiple windows and tabs" {
        val webDriver = driver ?: throw IllegalStateException("Driver not initialized")
        webDriver.get("https://the-internet.herokuapp.com/windows")
        // Store the original window handle
        val originalWindow = webDriver.windowHandle
        // Click the link which opens a new window
        webDriver.waitForElementVisible(By.cssSelector(".example a")).click()
        // Find the new window handle by iterating through all open windows
        var newWindow: String? = null
        for (windowHandle in webDriver.windowHandles) {
            if (originalWindow != windowHandle) {
                newWindow = windowHandle
                break
            }
        }
        // Switch context to the new window
        webDriver.switchTo().window(newWindow!!)
        // Verify we are in the new window
        val newWindowText = webDriver.waitForElementVisible(By.cssSelector("h3")).text
        newWindowText shouldBe "New Window"
        // Close the new window (do not use quit() here!)
        webDriver.close()
        // Switch context back to the original window
        webDriver.switchTo().window(originalWindow)
        // Verify we are back
        val originalText = webDriver.findElement(By.cssSelector("h3")).text
        originalText shouldBe "Opening a new window"
    }
})
```

### Key Takeaways

* **Store Your Origin:** Always save the original `windowHandle` before triggering an action that opens a new tab. This makes finding your way back trivial.
* **close() vs quit():** 
    * `driver.close()` shuts down the *current* window the driver is focused on. If it's the only window, the session ends.
    * `driver.quit()` violently shuts down *all* windows and entirely terminates the WebDriver session.
* **Selenium 4 Convenience:** In Selenium 4, you can also use `driver.switchTo().newWindow(WindowType.TAB)` to explicitly create and switch to a brand new tab without clicking a link.

### Execution Output

```
Initializing Headless Chrome Driver for CI/CD...
Waiting up to 10 seconds for element: By.cssSelector: .example a
Waiting up to 10 seconds for element: By.cssSelector: h3
Tests: 1, Passed: 1, Failed: 0
```

### Conclusion

Window handle iteration allows your test suite to span across multiple tabs just like a real user. Keep track of your handles, and you will never lose control of your browser session!

In our next blog, we will tackle a massive topic: **Introduction to the Page Object Model (POM) Design Pattern**. We'll learn how to structure robust, scalable test suites.
