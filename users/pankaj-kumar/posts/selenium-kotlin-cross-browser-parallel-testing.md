---
title: "Cross-Browser Parallel Testing in Selenium Kotlin using Kotest"
date: "2025-03-11"
description: "Slash your test suite execution time by refactoring your Driver Manager for cross-browser support and executing tests simultaneously across Chrome, Firefox, and Edge using Kotest's parallel engine."
tags: ["Selenium", "Kotlin", "Parallel Testing", "Cross Browser", "Kotest", "Performance"]
---

Welcome to Blog 34 of our Selenium Kotlin Mastery Series!

We are nearing the end of our journey. Currently, our test suite is powerful and stable, but it suffers from two flaws:
1. It hardcodes Google Chrome as the sole execution browser.
2. It executes tests sequentially (one after the other). If we had 1,000 tests, it would take hours!

Today, we will fix both flaws. We will refactor our `ThreadSafeDriverManager` to support Chrome, Firefox, and Edge, and we will unleash Kotest's multithreaded Coroutine engine to run them all at exactly the same time!

### Step 1: Upgrading the Driver Manager

Let's modify our `ThreadSafeDriverManager.kt` from Phase 2. We'll introduce a `browser` parameter to our `getDriver()` method, utilizing a `when` block to dynamically spin up the correct `WebDriver` implementation.

Notice how we maintain the `ThreadLocal` wrapper. This ensures that when 3 different browsers are running on 3 different CPU threads simultaneously, they never accidentally steal each other's sessions!

```kotlin
package com.mycodeyatra.utils
import org.openqa.selenium.WebDriver
import org.openqa.selenium.chrome.ChromeDriver
import org.openqa.selenium.chrome.ChromeOptions
import org.openqa.selenium.edge.EdgeDriver
import org.openqa.selenium.edge.EdgeOptions
import org.openqa.selenium.firefox.FirefoxDriver
import org.openqa.selenium.firefox.FirefoxOptions
object ThreadSafeDriverManager {
    private val driverLocal = ThreadLocal<WebDriver>()
    // Default to Chrome, but accept dynamic overrides
    fun getDriver(browser: String = System.getProperty("browser", "chrome")): WebDriver {
        if (driverLocal.get() == null) {
            println("Initializing Headless $browser Driver for Thread: ${Thread.currentThread().id}")
            val driver = when (browser.lowercase()) {
                "firefox" -> {
                    val options = FirefoxOptions()
                    options.addArguments("-headless")
                    FirefoxDriver(options)
                }
                "edge" -> {
                    val options = EdgeOptions()
                    options.addArguments("--headless=new")
                    EdgeDriver(options)
                }
                else -> {
                    val options = ChromeOptions()
                    options.addArguments("--headless=new")
                    ChromeDriver(options)
                }
            }
            driverLocal.set(driver)
        }
        return driverLocal.get()
    }
    fun quitDriver() {
        if (driverLocal.get() != null) {
            driverLocal.get().quit()
            driverLocal.remove()
        }
    }
}
```

### Step 2: Parallel Execution in Kotest

Kotest makes parallelization laughably easy. Simply set the `threads` property inside your Spec closure, and wrap your test block in a `forEach` loop across your target browsers.

```kotlin
package com.mycodeyatra.tests
import com.mycodeyatra.utils.ThreadSafeDriverManager
import io.kotest.core.spec.style.StringSpec
import io.kotest.matchers.shouldBe
class Blog34_CrossBrowserTest : StringSpec({
    // 1. Tell Kotest to run iterations of this spec in parallel across 3 threads
    threads = 3
    val browsers = listOf("chrome", "firefox", "edge")
    // 2. Data-Drive the browser targets
    browsers.forEach { browser ->
        "Verify Login Page Title on $browser" {
            // Pass the browser dynamically to the Driver Manager
            val driver = ThreadSafeDriverManager.getDriver(browser)
            try {
                driver.get("https://practice.mycodeyatra.com/#/login")
                driver.title shouldBe "MyCodeYatra | Test Automation Sandbox"
                println("Successfully executed on $browser!")
            } finally {
                ThreadSafeDriverManager.quitDriver()
            }
        }
    }
})
```

### Execution Output

Watch your terminal closely. You will notice that Chrome, Firefox, and Edge all initialize simultaneously on different Thread IDs!

```
[INFO] Running com.mycodeyatra.tests.Blog34_CrossBrowserTest
Initializing Headless firefox Driver for Thread: 45
Initializing Headless edge Driver for Thread: 46
Initializing Headless chrome Driver for Thread: 44
Successfully executed on firefox!
Successfully executed on chrome!
Successfully executed on edge!
Tests: 3, Passed: 3, Failed: 0
```

### Conclusion

By combining Data-Driven Testing (the browser list) with Kotest's `threads` property, we achieved instantaneous Cross-Browser testing. Your suite will now uncover browser-specific rendering bugs, while executing 3x faster!

In our 35th and final blog of the Kotlin series, we will explore **Advanced Selenium 4 Features**, including Chrome DevTools Protocol (CDP) and Network Interception!
