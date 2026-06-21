---
title: "Builder Pattern in Selenium Kotlin: Simplifying Dynamic Test Data Generation"
date: "2025-03-04"
description: "Discover why the traditional Java Builder Pattern is obsolete in Kotlin, and learn how to use Data Classes to dynamically generate test data for Selenium."
tags: ["Selenium", "Kotlin", "Builder Pattern", "Design Patterns", "Data Classes"]
---

In traditional Java automation frameworks, creating test data models is notoriously verbose. If you want to instantiate a `User` object where you only define the `username` but leave `password` and `role` as their default values, you usually have to implement the **Builder Design Pattern**. This requires writing an inner `Builder` class, returning `this` on every setter, and finishing with a `.build()` method.

In Kotlin, this entire pattern is obsolete.

In this 4th post of our **Framework Design & Patterns** series, we will see how Kotlin's native language features elegantly replace the Builder Pattern, making test data generation an absolute breeze for our [MyCodeYatra Practice Site](https://practice.mycodeyatra.com/#/sandbox).

### The Kotlin "Builder": Data Classes & Default Arguments

Instead of a bulky 50-line Builder class, Kotlin accomplishes the exact same thing in just 5 lines using a `data class` with default parameters.

```kotlin
package com.mycodeyatra.models
// The Builder Pattern is naturally replaced by Data Classes with default arguments!
data class TestUser(
    val username: String = "admin",
    val password: String = "admin123",
    val role: String = "Admin",
    val isActive: Boolean = true
)
```

By providing default values, you can instantiate this class in several ways:
* `TestUser()` gives you the default admin user.
* `TestUser(username = "guest")` gives you a user with the name "guest", but the password remains "admin123".

### Immutability with the copy() Method

What if you have a base user, and you just want to tweak one property for a specific test? Kotlin provides the `copy()` method for all data classes natively. This returns a brand new object, keeping your data strictly immutable and thread-safe.

Let's see this in action within our Kotest scripts:

```kotlin
package com.mycodeyatra.tests
import com.mycodeyatra.models.TestUser
import com.mycodeyatra.pages.Blog24_LoginPage
import com.mycodeyatra.utils.ThreadSafeDriverManager
import io.kotest.core.spec.style.StringSpec
import io.kotest.matchers.string.shouldContain
class Blog27_BuilderPatternTest : StringSpec({
    afterSpec {
        ThreadSafeDriverManager.quitDriver()
    }
    "Test Login using default Data Class Builder" {
        val webDriver = ThreadSafeDriverManager.getDriver()
        webDriver.get("https://practice.mycodeyatra.com/#/login")
        // Using the default user which defaults to admin / admin123
        val defaultUser = TestUser()
        val loginPage = Blog24_LoginPage(webDriver)
        loginPage.enterUsername(defaultUser.username)
        loginPage.enterPassword(defaultUser.password)
        loginPage.clickLogin()
        loginPage.getProfileTitle() shouldContain "Welcome back, ${defaultUser.username}"
    }
    "Test Login using modified Data Class copy" {
        val webDriver = ThreadSafeDriverManager.getDriver()
        webDriver.get("https://practice.mycodeyatra.com/#/login")
        // The Kotlin way of the Builder pattern: use copy() to change only what you need
        val invalidUser = TestUser().copy(username = "hacker", password = "wrongpassword")
        val loginPage = Blog24_LoginPage(webDriver)
        loginPage.enterUsername(invalidUser.username)
        loginPage.enterPassword(invalidUser.password)
        loginPage.clickLogin()
        loginPage.getErrorMessage() shouldContain "Invalid username or password"
    }
})
```

### Execution Output

```
[INFO] Running com.mycodeyatra.tests.Blog27_BuilderPatternTest
Initializing new Headless Chrome Driver for Thread: 1
Quitting Driver for Thread: 1
Tests: 2, Passed: 2, Failed: 0
```

### Conclusion

Kotlin completely eradicates the boilerplate required for the Builder pattern. Data Classes combined with named parameters and the `copy()` method provide an immutable, thread-safe, and highly readable way to inject dynamic data into your Page Objects.

In our next blog, we will take our framework a step further by implementing the **Screenplay Pattern**, an advanced evolution of the Page Object Model designed around the SOLID principles!
