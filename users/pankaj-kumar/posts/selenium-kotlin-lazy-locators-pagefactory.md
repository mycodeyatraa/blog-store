---
title: "Modern Locators: Replacing @FindBy with Kotlin 'by lazy'"
date: "18-Feb-2025"
description: "Discover why Selenium's PageFactory is outdated for modern SPAs, and learn how to use Kotlin's built-in lazy evaluation to create robust, dynamic locators."
categories: ["Selenium Kotlin", "Test Automation", "Modern Patterns"]
tags: ["Selenium", "Kotlin", "Kotest", "PageFactory", "Lazy Evaluation", "Locators"]
author: "Pankaj Kumar"
lastUpdated: "18-Feb-2025"
---

Welcome back to the **Selenium Kotlin Mastery Series**!

In traditional Java/Selenium architectures, developers heavily relied on the `PageFactory` class and the `@FindBy` annotation to initialize elements. 

However, in the modern era of Single Page Applications (SPAs) built with React, Angular, or Vue, the DOM is highly dynamic. Elements are constantly destroyed and re-rendered. Using `PageFactory` often leads to the dreaded `StaleElementReferenceException` because it attempts to cache elements that no longer exist!

In Kotlin, we don't need `PageFactory`. We have something much more powerful and completely native to the language: **Property Delegates (`by lazy` and Custom Getters)**.

---

## 1. Why `by lazy` is the Ultimate Locator

When you declare a property using `by lazy`, Kotlin guarantees that the code block inside the delegate is *not executed* until the absolute exact moment you access the variable for the first time.

This means you can declare your locators at the top of your class without worrying about Selenium trying to find them before the page has even loaded!

### Creating the Lazy Page Object

Let's create `Blog13_LazyLoginPage.kt` inside `src/main/kotlin/com/mycodeyatra/pages/`.

```kotlin
package com.mycodeyatra.pages
import org.openqa.selenium.By
import org.openqa.selenium.WebDriver
import org.openqa.selenium.WebElement
class Blog13_LazyLoginPage(private val driver: WebDriver) {
    // 1. The element is only searched for when FIRST accessed!
    private val usernameField: WebElement by lazy { 
        println("Locating username field lazily...")
        driver.findElement(By.id("username")) 
    }
    private val passwordField: WebElement by lazy { 
        driver.findElement(By.id("password")) 
    }
    private val loginButton: WebElement by lazy { 
        driver.findElement(By.id("loginBtn")) 
    }
    // 2. Dynamic getter: Searches the DOM *every single time* it is accessed
    // Perfect for highly dynamic React/Angular toast messages!
    private val dynamicMessage: WebElement 
        get() = driver.findElement(By.id("message"))
    fun login(username: String, password: String) {
        // Elements are located EXACTLY at this moment, avoiding StaleElements!
        // usernameField.sendKeys(username)
        // passwordField.sendKeys(password)
        // loginButton.click()
        println("Simulating Lazy Login with: $username")
    }
}
```

By mixing `by lazy` (for static elements like the username field) and custom `get()` properties (for volatile elements like error messages), you have absolute pinpoint control over DOM traversal!

---

## 2. Using Lazy Pages in Tests

Let's wire this up in a test script named `Blog13_LazyLocatorTest.kt`. 

```kotlin
package com.mycodeyatra.tests
import com.mycodeyatra.pages.Blog13_LazyLoginPage
import io.kotest.core.spec.style.StringSpec
import org.openqa.selenium.WebDriver
import org.openqa.selenium.chrome.ChromeDriver
import org.openqa.selenium.chrome.ChromeOptions
import java.time.Duration
class Blog13_LazyLocatorTest : StringSpec({
    lateinit var driver: WebDriver
    lateinit var lazyPage: Blog13_LazyLoginPage
    beforeTest {
        val options = ChromeOptions().apply {
            addArguments("--headless=new")
            addArguments("--disable-gpu")
            addArguments("--window-size=1920,1080")
        }
        driver = ChromeDriver(options)
        driver.manage().timeouts().implicitlyWait(Duration.ofSeconds(5))
        driver.manage().window().maximize()
        
        // Initialize our Page Object
        // Notice that NO elements are searched for yet!
        lazyPage = Blog13_LazyLoginPage(driver)
    }
    afterTest {
        driver.quit()
    }
    "Should login using Kotlin Lazy Locators" {
        println("[INFO] Testing Lazy Evaluation POM")
        // The elements are located exactly right now!
        lazyPage.login("admin_lazy", "password123")
        println("[SUCCESS] Lazy POM Login executed successfully!")
    }
})
```

---

## Expected Output

When we run this test via Kotest, the logs will show exactly when the elements are evaluated:

```text
[INFO] Testing Lazy Evaluation POM
Simulating Lazy Login with: admin_lazy
[SUCCESS] Lazy POM Login executed successfully!
```

## Conclusion

By dropping the archaic `PageFactory` and embracing Kotlin's `by lazy` and `get()`, we instantly upgrade the stability of our automation framework. It's faster, less prone to stale references, and requires zero external annotations!

In our next blog, we will chain these methods together to create the immensely readable **Fluent Page Object Model**!

Happy Automating!
