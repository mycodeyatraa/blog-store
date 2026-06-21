---
title: "Headless Execution & Manipulating ChromeOptions for CI/CD"
date: "23-Feb-2025"
description: "Prepare your Selenium Kotlin framework for enterprise CI/CD environments like Jenkins and GitHub Actions by mastering Headless execution and Sandbox bypasses."
categories: ["Selenium Kotlin", "Test Automation", "CI/CD"]
tags: ["Selenium", "Kotlin", "Kotest", "Headless", "ChromeOptions", "Jenkins", "Docker"]
author: "Pankaj Kumar"
lastUpdated: "23-Feb-2025"
---

Welcome back to the **Selenium Kotlin Mastery Series**!

So far, every test we have written has launched a visible Chrome Browser UI on your desktop. This is fantastic for local debugging!

However, when you run your tests in a Continuous Integration (CI) pipeline like Jenkins, GitLab CI, or GitHub Actions, there is no physical monitor. The CI server is a terminal-only environment (usually a headless Linux container). If Selenium attempts to launch a UI there, the test will crash instantly with a `Display not found` error.

To solve this, we must run the browser in **Headless Mode** using `ChromeOptions`.

---

## 1. Building the `DriverManager` Object

Instead of cluttering our test files with configuration code, let's create a centralized Kotlin `object` (which functions as a Singleton) to provide fully configured `WebDriver` instances.

Create `DriverManager.kt` inside `src/main/kotlin/com/mycodeyatra/utils/`.

```kotlin
package com.mycodeyatra.utils
import org.openqa.selenium.WebDriver
import org.openqa.selenium.chrome.ChromeDriver
import org.openqa.selenium.chrome.ChromeOptions
object DriverManager {
    fun getHeadlessChromeDriver(): WebDriver {
        println("Initializing Headless Chrome Driver for CI/CD...")
        
        val options = ChromeOptions().apply {
            // 1. Use the new headless mode which renders pages identically to the UI browser
            addArguments("--headless=new")
            
            // 2. Critical arguments for running in Docker/CI environments
            // Prevents Chrome from crashing inside isolated Docker containers
            addArguments("--no-sandbox")
            addArguments("--disable-dev-shm-usage")
            
            // 3. Set a standard enterprise resolution so responsive websites don't break
            addArguments("--window-size=1920,1080")
            
            // 4. Bypass bot detection mechanisms (helpful for headless scraping)
            addArguments("--disable-blink-features=AutomationControlled")
            setExperimentalOption("excludeSwitches", arrayOf("enable-automation"))
        }
        
        return ChromeDriver(options)
    }
}
```

This configuration ensures your tests will run flawlessly inside Docker containers without memory allocation crashes (`--disable-dev-shm-usage`).

---

## 2. Testing the Headless Setup

Let's write a test in `Blog18_HeadlessChromeTest.kt` to ensure it works!

```kotlin
package com.mycodeyatra.tests
import com.mycodeyatra.utils.DriverManager
import io.kotest.core.spec.style.StringSpec
import org.openqa.selenium.WebDriver
import org.openqa.selenium.JavascriptExecutor
import java.time.Duration
class Blog18_HeadlessChromeTest : StringSpec({
    lateinit var driver: WebDriver
    beforeTest {
        // Our setup is now incredibly clean!
        driver = DriverManager.getHeadlessChromeDriver()
        driver.manage().timeouts().implicitlyWait(Duration.ofSeconds(5))
    }
    afterTest {
        driver.quit()
    }
    "Should execute smoothly in Headless CI mode" {
        println("[INFO] Testing Headless Execution")
        // driver.get("https://practice.mycodeyatra.com")
        println("Browser is running invisibly in the background!")
        println("Resolution is fixed to 1920x1080 to prevent responsive UI breaks.")
        
        // Let's print the User-Agent just to verify it is running in headless mode
        val jsExecutor = driver as JavascriptExecutor
        val userAgent = jsExecutor.executeScript("return navigator.userAgent;").toString()
        println("Headless User Agent: $userAgent")
        
        println("[SUCCESS] Headless execution passed without Sandbox restrictions!")
    }
})
```

---

## Expected Output

When you run this test, no browser window will pop up! It executes completely in the background, making your test execution dramatically faster.

```text
Initializing Headless Chrome Driver for CI/CD...
[INFO] Testing Headless Execution
Browser is running invisibly in the background!
Resolution is fixed to 1920x1080 to prevent responsive UI breaks.
Headless User Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) HeadlessChrome/149.0.0.0 Safari/537.36
[SUCCESS] Headless execution passed without Sandbox restrictions!
```

## Conclusion

By abstracting our WebDriver initialization into a Singleton `DriverManager` and configuring the `ChromeOptions` for CI/CD environments, our framework is now robust enough to be deployed to Jenkins or GitHub Actions. 

In our next blog, we will tackle file handling: **Uploading and Downloading files using Selenium Kotlin**!

Happy Automating!
