---
title: "Capturing Screenshots during Test Execution in Selenium Kotlin"
date: "2025-02-25"
description: "Learn how to capture crucial visual evidence during automation. This post covers full-page and element-level screenshots using Selenium Kotlin."
tags: ["Selenium", "Kotlin", "Screenshots", "Automation", "Kotest"]
---

When tests fail in CI/CD pipelines, diagnosing the root cause from a console stack trace can be like finding a needle in a haystack. This is where screenshots become invaluable. Capturing the visual state of the browser at the moment of failure gives you immediate context.

In this 20th post of our Selenium Kotlin Mastery Series, we will extend our `WebDriverExtensions` to effortlessly capture both full-viewport and element-specific screenshots.

### The Power of Extension Functions

Selenium's `TakesScreenshot` interface is powerful but requires type casting in Java/Kotlin which can make our test scripts look messy. Instead of doing the cast inside every test method, we can abstract this into Kotlin extension functions.

Let's update our `WebDriverExtensions.kt` to include these utilities:

```kotlin
package com.mycodeyatra.utils

import org.openqa.selenium.By
import org.openqa.selenium.OutputType
import org.openqa.selenium.TakesScreenshot
import org.openqa.selenium.WebDriver
import org.openqa.selenium.WebElement
import java.io.File

// ... Existing Wait Extensions ...

// Extension to take a screenshot of the entire visible viewport
fun WebDriver.takeScreenshot(destinationFile: File) {
    println("Taking full page screenshot and saving to: ${destinationFile.absolutePath}")
    val screenshotFile = (this as TakesScreenshot).getScreenshotAs(OutputType.FILE)
    screenshotFile.copyTo(destinationFile, overwrite = true)
}

// Extension to take a screenshot of a specific element only
fun WebElement.takeElementScreenshot(destinationFile: File) {
    println("Taking element screenshot and saving to: ${destinationFile.absolutePath}")
    val screenshotFile = this.getScreenshotAs(OutputType.FILE)
    screenshotFile.copyTo(destinationFile, overwrite = true)
}
```

### Implementing the Screenshot Tests

Now let's write Kotest specifications to demonstrate both features. We will use a temporary directory provided by Kotlin's standard library to ensure our file system isn't cluttered after the test executes.

```kotlin
package com.mycodeyatra.tests

import com.mycodeyatra.utils.DriverManager
import com.mycodeyatra.utils.takeElementScreenshot
import com.mycodeyatra.utils.takeScreenshot
import com.mycodeyatra.utils.waitForElementVisible
import io.kotest.core.spec.style.StringSpec
import io.kotest.matchers.file.shouldExist
import org.openqa.selenium.By
import org.openqa.selenium.WebDriver
import java.io.File
import kotlin.io.path.createTempDirectory

class Blog20_ScreenshotsTest : StringSpec({

    var driver: WebDriver? = null
    lateinit var screenshotDir: File

    beforeSpec {
        // Create a temporary directory for our screenshots
        screenshotDir = createTempDirectory("selenium-screenshots").toFile()
        driver = DriverManager.getHeadlessChromeDriver()
        driver?.manage()?.window()?.maximize()
    }

    afterSpec {
        driver?.quit()
        // Clean up our temporary directory
        screenshotDir.deleteRecursively()
    }

    "Take a full viewport screenshot" {
        val webDriver = driver ?: throw IllegalStateException("Driver not initialized")
        
        webDriver.get("https://the-internet.herokuapp.com/login")
        
        val fullScreenshotFile = File(screenshotDir, "full_viewport.png")
        webDriver.takeScreenshot(fullScreenshotFile)
        
        // Assert the file was actually created
        fullScreenshotFile.shouldExist()
        println("Full screenshot size: ${fullScreenshotFile.length()} bytes")
    }

    "Take an element-level screenshot" {
        val webDriver = driver ?: throw IllegalStateException("Driver not initialized")
        
        webDriver.get("https://the-internet.herokuapp.com/login")
        
        // Wait for the login form to be visible
        val loginForm = webDriver.waitForElementVisible(By.id("login"))
        
        val elementScreenshotFile = File(screenshotDir, "login_form.png")
        loginForm.takeElementScreenshot(elementScreenshotFile)
        
        // Assert the element screenshot was created
        elementScreenshotFile.shouldExist()
        println("Element screenshot size: ${elementScreenshotFile.length()} bytes")
    }
})
```

### Execution Output

When you run this script, Selenium takes the snapshot and saves it as a PNG file automatically. 

```
Initializing Headless Chrome Driver for CI/CD...
Taking full page screenshot and saving to: /tmp/selenium-screenshots1234/full_viewport.png
Full screenshot size: 45210 bytes
Waiting up to 10 seconds for element: By.id: login
Taking element screenshot and saving to: /tmp/selenium-screenshots1234/login_form.png
Element screenshot size: 12050 bytes

Tests: 2, Passed: 2, Failed: 0
```

### Best Practices

1. **Failure Listener:** In a real framework, you should integrate `takeScreenshot` inside an `afterTest` listener. This ensures screenshots are *only* taken when an assertion fails, saving disk space.
2. **Naming Conventions:** Inject the test name and a timestamp into the screenshot filename. This makes correlating screenshots to failed CI runs much easier.
3. **Element vs Full Page:** Element screenshots are excellent for visual regression testing (VRT), while full-page screenshots are better for debugging functional test failures.

### Conclusion

With Kotlin extension functions, taking screenshots becomes a one-liner that can be invoked on either the `WebDriver` itself or a specific `WebElement`. By combining this with our existing wait wrappers, our framework is rapidly maturing into a robust, enterprise-grade solution.

In our next blog post, we will explore **Advanced Interactions: Mouse Hover and Drag-and-Drop using the Actions Class**!
