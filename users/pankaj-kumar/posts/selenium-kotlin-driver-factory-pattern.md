---
title: "Driver Factory Pattern in Selenium Kotlin: Designing a Thread-Safe Singleton"
date: "2025-03-03"
description: "Master the Driver Factory design pattern in Kotlin using ThreadLocal to achieve thread-safe WebDriver instantiation for parallel test execution."
tags: ["Selenium", "Kotlin", "Parallel Execution", "ThreadLocal", "Design Patterns", "Kotest"]
---

Up until now, our test classes have been manually instantiating a WebDriver in their `beforeSpec` blocks and storing it in a global class-level variable like `var driver: WebDriver? = null`. 

This approach is perfectly fine for sequential execution. However, the moment you configure Kotest to run tests in **parallel** across multiple CPU threads, this design will catastrophically fail. Multiple threads will overwrite your global `driver` variable, causing browsers to randomly close and tests to step on each other's toes.

In this 3rd post of our **Framework Design & Patterns** series, we will implement the **Driver Factory Pattern** combined with a **Thread-Safe Singleton** using Kotlin's `ThreadLocal`.

### Understanding ThreadLocal

`ThreadLocal` is a construct that allows you to store data that will only be accessible by a specific thread. If Thread A and Thread B both call `ThreadLocal.get()`, they will receive completely different objects. 

This is the magic bullet for parallel execution in Selenium. It guarantees that each running test thread gets its own isolated browser window.

### Step 1: Building the ThreadSafeDriverManager

Let's create a new utility object in Kotlin. We will use a `ThreadLocal<WebDriver>` to hold our driver instances.

```kotlin
package com.mycodeyatra.utils
import org.openqa.selenium.WebDriver
import org.openqa.selenium.chrome.ChromeDriver
import org.openqa.selenium.chrome.ChromeOptions
object ThreadSafeDriverManager {
    // ThreadLocal ensures each thread running a test gets its own isolated WebDriver
    private val driverThreadLocal = ThreadLocal<WebDriver>()
    fun getDriver(): WebDriver {
        if (driverThreadLocal.get() == null) {
            println("Initializing new Headless Chrome Driver for Thread: ${Thread.currentThread().id}")
            val options = ChromeOptions().apply {
                addArguments("--headless=new")
                addArguments("--no-sandbox")
                addArguments("--disable-dev-shm-usage")
                addArguments("--window-size=1920,1080")
            }
            driverThreadLocal.set(ChromeDriver(options))
        }
        return driverThreadLocal.get()
    }
    fun quitDriver() {
        driverThreadLocal.get()?.let {
            println("Quitting Driver for Thread: ${Thread.currentThread().id}")
            it.quit()
        }
        // Critical: Remove the reference to prevent memory leaks in thread pools!
        driverThreadLocal.remove()
    }
}
```

### Step 2: Refactoring Our Tests

With our `ThreadSafeDriverManager` in place, we no longer need to hold state in our test classes. Whenever a test needs the driver, it simply calls `getDriver()`.

```kotlin
package com.mycodeyatra.tests
import com.mycodeyatra.pages.Blog24_LoginPage
import com.mycodeyatra.utils.ThreadSafeDriverManager
import io.kotest.core.spec.style.StringSpec
import io.kotest.matchers.shouldBe
import io.kotest.matchers.string.shouldContain
class Blog26_ThreadSafeTest : StringSpec({
    // We do NOT store 'var driver: WebDriver? = null' as a class property anymore!
    afterSpec {
        ThreadSafeDriverManager.quitDriver()
    }
    "Test 1: Login should succeed on Thread 1" {
        // Fetch the driver dedicated to this current thread
        val webDriver = ThreadSafeDriverManager.getDriver()
        webDriver.get("https://practice.mycodeyatra.com/#/login")
        val loginPage = Blog24_LoginPage(webDriver)
        loginPage.enterUsername("admin")
        loginPage.enterPassword("admin123")
        loginPage.clickLogin()
        loginPage.getProfileTitle() shouldContain "Welcome back, admin"
    }
    "Test 2: Driver identity should remain constant across the same thread" {
        val firstCall = ThreadSafeDriverManager.getDriver()
        val secondCall = ThreadSafeDriverManager.getDriver()
        // Assert that both calls return the exact same object reference in memory
        (firstCall === secondCall) shouldBe true
    }
})
```

### Execution Output

```
[INFO] Running com.mycodeyatra.tests.Blog26_ThreadSafeTest
Initializing new Headless Chrome Driver for Thread: 1
Quitting Driver for Thread: 1
Tests: 2, Passed: 2, Failed: 0
```

Notice how `Initializing new Headless Chrome Driver` was only printed once, even though we have two tests? Because both tests ran on `Thread: 1`, the Singleton pattern returned the already-active browser session for the second test!

### Conclusion

By moving our driver initialization into a `ThreadLocal` object, we have completely decoupled driver state from our test classes. Our framework is now fully prepared to handle high-speed parallel execution. 

In our next blog, we will explore the **Builder Pattern** for dynamically generating complex test data!
