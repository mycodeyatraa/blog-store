---
title: "Data-Driven Testing in Selenium Kotlin using Kotest Dynamic Tests"
date: "2025-03-02"
description: "Learn how to implement powerful Data-Driven Testing (DDT) in Selenium Kotlin by leveraging Kotlin collections and Kotest dynamic test generation."
tags: ["Selenium", "Kotlin", "Kotest", "DDT", "Data-Driven Testing", "Automation"]
---

Imagine you need to verify a login page against ten different combinations of invalid usernames and passwords. Writing ten separate test blocks would violate the DRY (Don't Repeat Yourself) principle and clutter your test file immensely.

In Java, you might use TestNG `@DataProvider` annotations or Apache POI for Excel integration. However, Kotlin combined with Kotest offers a vastly more elegant solution: **Dynamic Test Generation using standard Kotlin Collections.**

In this 25th post of our Selenium Kotlin Mastery Series, we will implement Data-Driven Testing (DDT) on the [MyCodeYatra Practice Login Page](https://practice.mycodeyatra.com/#/login).

### What is Data-Driven Testing?

Data-Driven Testing (DDT) is an execution strategy where a single test script is executed iteratively, passing a different set of test data in each iteration. It cleanly separates the test data from the test logic.

### Implementing DDT with Kotest

With Kotest, we don't need complex annotations. Since a `StringSpec` is essentially a Lambda function, we can define a standard Kotlin `List` of datasets and loop over it using `.forEach { }`. Kotest will intercept this loop and dynamically register a new, independent test case for every iteration!

Let's look at the implementation:

```kotlin
package com.mycodeyatra.tests
import com.mycodeyatra.pages.Blog24_LoginPage
import com.mycodeyatra.utils.DriverManager
import io.kotest.core.spec.style.StringSpec
import io.kotest.matchers.string.shouldContain
import org.openqa.selenium.WebDriver
class Blog25_DataDrivenTest : StringSpec({
    var driver: WebDriver? = null
    beforeSpec {
        driver = DriverManager.getHeadlessChromeDriver()
        driver?.manage()?.window()?.maximize()
    }
    afterSpec {
        driver?.quit()
    }
    // A list of test data sets: Pair of Username to Password
    val invalidLoginData = listOf(
        Pair("wrongUser", "wrongPass"),
        Pair("admin", "wrongPass"),
        Pair("wrongUser", "admin123"),
        Pair("", "")
    )
    // Data-Driven Test loop: Dynamically creates a test case for each dataset
    invalidLoginData.forEach { (username, password) ->
        "Login should fail for invalid credentials: '$username' / '$password'" {
            val webDriver = driver ?: throw IllegalStateException("Driver not initialized")
            webDriver.get("https://practice.mycodeyatra.com/#/login")
            val loginPage = Blog24_LoginPage(webDriver)
            loginPage.enterUsername(username)
            loginPage.enterPassword(password)
            loginPage.clickLogin()
            val errorText = loginPage.getErrorMessage()
            errorText shouldContain "Invalid username or password"
        }
    }
    "Login should succeed for valid credentials" {
        val webDriver = driver ?: throw IllegalStateException("Driver not initialized")
        webDriver.get("https://practice.mycodeyatra.com/#/login")
        val loginPage = Blog24_LoginPage(webDriver)
        loginPage.enterUsername("admin")
        loginPage.enterPassword("admin123")
        loginPage.clickLogin()
        val successMessage = loginPage.getProfileTitle()
        successMessage shouldContain "Welcome back, admin"
    }
})
```

### Why is this approach superior?

1. **No External Dependencies:** We used pure Kotlin constructs (`listOf`, `Pair`, `forEach`) without needing external CSV readers or complex DataProvider classes.
2. **Dynamic Test Names:** By interpolating the data directly into the test name string (`"Login should fail for invalid credentials: '$username'"`), your test reports will explicitly tell you exactly *which* dataset failed.
3. **Reusability:** The test data list (`invalidLoginData`) can easily be extracted into a separate `TestData.kt` file if it grows too large, keeping your test classes immaculate.

### Execution Output

When you run this file, Kotest will output 5 completely distinct tests (4 failing permutations and 1 success permutation):

```
[INFO] Running com.mycodeyatra.tests.Blog25_DataDrivenTest
Initializing Headless Chrome Driver for CI/CD...
Login should fail for invalid credentials: 'wrongUser' / 'wrongPass' (Passed)
Login should fail for invalid credentials: 'admin' / 'wrongPass' (Passed)
Login should fail for invalid credentials: 'wrongUser' / 'admin123' (Passed)
Login should fail for invalid credentials: '' / '' (Passed)
Login should succeed for valid credentials (Passed)
Tests: 5, Passed: 5, Failed: 0
```

### Conclusion

Data-Driven Testing in Kotlin is breathtakingly simple compared to legacy Java approaches. Leveraging standard collection loops empowers you to maximize test coverage with minimal lines of code.

In our next blog, we will cover the **Driver Factory Design Pattern**, exploring how to make our WebDriver instantiation Thread-Safe for parallel execution!
