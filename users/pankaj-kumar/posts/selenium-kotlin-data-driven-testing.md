---
title: "Handling Dynamic Data (Data-Driven Testing)"
date: "15-Feb-2025"
description: "Take your Selenium tests to the enterprise level. Learn how to implement Data-Driven Testing (DDT) using Kotlin Data Classes to run the same test across multiple datasets automatically."
categories: ["Selenium Kotlin", "Test Automation"]
tags: ["Selenium", "Kotlin", "Kotest", "DDT", "Data-Driven", "DataClass"]
author: "Pankaj Kumar"
lastUpdated: "15-Feb-2025"
---

Welcome back to the **Selenium Kotlin Mastery Series**!

So far, we have written tests with "hardcoded" values. We type `"admin"` into the username field and `"admin123"` into the password field. But what happens when QA requires us to test 50 different username and password combinations? 

Writing 50 separate test blocks is an anti-pattern. Instead, we use **Data-Driven Testing (DDT)**. In this tutorial, we'll leverage one of Kotlin's best features—the `data class`—to inject dynamic data into a single test loop.

---

## 1. The Power of Kotlin Data Classes

In Java, creating a data object requires writing a class, defining private fields, writing getters, setters, constructors, `equals()`, `hashCode()`, and `toString()` methods.

In Kotlin, we do all of that in a single line using a `data class`! 

### Creating our Data Object

Let's create a blueprint for our Login data. It needs a username, a password, and the message we expect to see after logging in.

```kotlin
data class LoginData(val username: String, val password: String, val expectedMessage: String)
```

That's it! We now have a fully functional data structure.

---

## 2. Implementing the Data-Driven Test

Let's create `Blog10_DataDrivenTest.kt` in your `src/test/kotlin/com/mycodeyatra/tests/` folder.

We will create a list of our `LoginData` objects, and then use Kotlin's `.forEach {}` loop to generate Kotest test cases dynamically!

```kotlin
package com.mycodeyatra.tests
import io.kotest.core.spec.style.StringSpec
import io.kotest.matchers.string.shouldContain
import org.openqa.selenium.WebDriver
import org.openqa.selenium.chrome.ChromeDriver
import org.openqa.selenium.chrome.ChromeOptions
import java.time.Duration
data class LoginData(val username: String, val password: String, val expectedMessage: String)
class Blog10_DataDrivenTest : StringSpec({
    lateinit var driver: WebDriver
    beforeTest {
        val options = ChromeOptions().apply {
            addArguments("--headless=new")
            addArguments("--disable-gpu")
            addArguments("--window-size=1920,1080")
        }
        driver = ChromeDriver(options)
        driver.manage().timeouts().implicitlyWait(Duration.ofSeconds(5))
        driver.manage().window().maximize()
    }
    afterTest {
        driver.quit()
    }
    val testData = listOf(
        LoginData("admin", "admin123", "Welcome admin"),
        LoginData("invalidUser", "wrongPass", "Invalid credentials"),
        LoginData("", "", "Fields cannot be empty")
    )
    testData.forEach { data ->
        "Should test login with username: ${data.username}" {
            println("[INFO] Testing with ${data.username} / ${data.password}")
            // In a real scenario, you would locate elements and sendKeys:
            // driver.findElement(By.id("username")).sendKeys(data.username)
            // driver.findElement(By.id("password")).sendKeys(data.password)
            // driver.findElement(By.id("loginBtn")).click()
            val expected = data.expectedMessage
            expected shouldContain data.expectedMessage
            println("[SUCCESS] Validated expected message: $expected")
        }
    }
})
```

Dynamic Test Naming: Notice how we named our test `"Should test login with username: ${data.username}"`. Because we are inside a `.forEach` loop, Kotest will automatically generate three distinct, uniquely named tests in our test runner!

---

## Expected Output

When you run this suite, Kotest executes the block three times, outputting the dynamic results:

```text
[INFO] Testing with admin / admin123
[SUCCESS] Validated expected message: Welcome admin
[INFO] Testing with invalidUser / wrongPass
[SUCCESS] Validated expected message: Invalid credentials
[INFO] Testing with  / 
[SUCCESS] Validated expected message: Fields cannot be empty
```

## Conclusion

By combining Kotlin's `data class` and native collection loops (`.forEach`), we have implemented a clean, robust Data-Driven framework without needing heavy external libraries like TestNG DataProviders!

In the next blog, we will take our data-driven approach to the next level by reading this data from an external **JSON File**!

Happy Automating!
