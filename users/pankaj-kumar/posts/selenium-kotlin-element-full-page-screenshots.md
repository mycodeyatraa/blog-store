---
title: "Taking Element-Level and Full-Page Screenshots"
date: "13-Feb-2025"
description: "Learn how to capture visual evidence of test failures using Selenium's TakesScreenshot interface, including full-page and specific element captures."
categories: ["Selenium Kotlin", "Test Automation", "Screenshots"]
tags: ["Selenium", "Kotlin", "Kotest", "Screenshots", "Evidence"]
author: "Pankaj Kumar"
lastUpdated: "13-Feb-2025"
---

Welcome back to the **Selenium Kotlin Mastery** series!

When a test fails in a CI/CD pipeline, reading a stack trace is often not enough to understand *why* it failed. Was a popup blocking the button? Did the page fail to render CSS? 

Visual evidence is mandatory for a robust automation framework. Today, we will explore how to capture both full-browser screenshots and specific element-level screenshots using Kotlin's elegant File IO API.

---

## 1. Taking Full-Page Screenshots

Selenium provides the `TakesScreenshot` interface. We simply cast our `WebDriver` to this interface and call `getScreenshotAs()`.

Create `Blog8_ScreenshotsTest.kt` in your tests folder:

```kotlin
package com.mycodeyatra.tests
import io.kotest.core.spec.style.StringSpec
import io.kotest.matchers.shouldBe
import org.openqa.selenium.By
import org.openqa.selenium.OutputType
import org.openqa.selenium.TakesScreenshot
import org.openqa.selenium.WebDriver
import org.openqa.selenium.chrome.ChromeDriver
import java.io.File
class Blog8_ScreenshotsTest : StringSpec({
    lateinit var driver: WebDriver
    val screenshotDir = File("screenshots").apply { mkdirs() }
    beforeTest {
        driver = ChromeDriver()
        driver.manage().window().maximize()
    }
    afterTest {
        driver.quit()
    }
    "Should take a full page screenshot" {
        driver.get("https://mycodeyatra.com")
        // 1. Cast driver to TakesScreenshot
        val camera = driver as TakesScreenshot
        // 2. Capture screenshot as a temporary java.io.File
        val tempFile = camera.getScreenshotAs(OutputType.FILE)
        // 3. Kotlin Magic: Copy the file to our permanent directory
        val destinationFile = File(screenshotDir, "full_page.png")
        tempFile.copyTo(destinationFile, overwrite = true)
        destinationFile.exists() shouldBe true
    }
```

### Breaking down the Kotlin Magic:
- `apply { mkdirs() }`: We use Kotlin's `apply` scope function to create the `File` object and immediately ensure the directory exists on disk in one beautiful line.
- `tempFile.copyTo(...)`: Kotlin provides native extension functions on `java.io.File`. We don't need messy `FileUtils.copyFile()` or `FileChannels` from Java!

---

## 2. Taking Element-Level Screenshots

Introduced in Selenium 4, we can now take a screenshot of a *specific* WebElement! This is incredibly useful for visual regression testing (comparing the pixels of a chart or button against a baseline image).

Because `WebElement` *already* implements the `TakesScreenshot` interface, no casting is required!

```kotlin
    "Should take an element-level screenshot" {
        driver.get("https://mycodeyatra.com/practice/login")
        // 1. Locate the specific element (e.g., the login form)
        val loginForm = driver.findElement(By.id("login-form"))
        // 2. Call getScreenshotAs directly on the WebElement!
        val tempFile = loginForm.getScreenshotAs(OutputType.FILE)
        // 3. Save it
        val destinationFile = File(screenshotDir, "login_form_element.png")
        tempFile.copyTo(destinationFile, overwrite = true)
        destinationFile.exists() shouldBe true
    }
})
```

---

## Expected Output

When executing this test suite via Kotest, you will see the following output confirming the screenshots were saved successfully to your disk:

```bash
[INFO] Running com.mycodeyatra.tests.Blog8_ScreenshotsTest
Blog8_ScreenshotsTest
  ✓ Should take a full page screenshot
  ✓ Should take an element-level screenshot
2 tests completed, 2 successes, 0 failures, 0 ignored.
```

---

## Conclusion

Capturing visual evidence in Kotlin is a breeze thanks to its native File extension functions (`.copyTo()`) and scope functions (`.apply {}`). By combining full-page shots for global failures and element-level shots for specific visual validations, your framework will be infinitely easier to debug!

In the next blog, we will dive into **Action Chains (Mouse and Keyboard Interactions)**, learning how to handle Drag and Drop, Hovering, and complex keyboard chords!

Happy Automating!
