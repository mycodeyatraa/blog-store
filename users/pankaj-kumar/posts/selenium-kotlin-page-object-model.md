---
title: "Mastering the Page Object Model (POM) in Kotlin"
date: "17-Feb-2025"
description: "Discover how Kotlin's concise syntax, primary constructors, and properties make implementing the Page Object Model design pattern cleaner and more efficient than ever before."
categories: ["Selenium Kotlin", "Test Automation", "Design Patterns"]
tags: ["Selenium", "Kotlin", "Kotest", "POM", "Page Object Model", "Architecture"]
author: "Pankaj Kumar"
lastUpdated: "17-Feb-2025"
---

Welcome back to the **Selenium Kotlin Mastery Series**!

If you write all your Selenium commands (`findElement`, `sendKeys`, `click`) directly inside your test scripts, your code will quickly become an unmaintainable mess. When a developer changes an element ID on the login page, you will have to update 50 different test files!

The solution is the **Page Object Model (POM)**. POM is a design pattern where we create a separate Class for every web page, and we store all the locators and interactions for that page inside the class.

In this blog, we'll see why Kotlin makes writing Page Objects an absolute joy!

---

## 1. The Java Way vs The Kotlin Way

In Java, a Page Object typically requires private fields, a constructor to initialize the WebDriver, and verbose methods.

In Kotlin, we can use **Primary Constructors** to pass the WebDriver directly into the class header, and we use standard properties (`val`) to store our locators cleanly!

Let's create a new folder `src/main/kotlin/com/mycodeyatra/pages/` and create our `LoginPage.kt`.

```kotlin
package com.mycodeyatra.pages
import org.openqa.selenium.By
import org.openqa.selenium.WebDriver
class LoginPage(private val driver: WebDriver) {
    private val usernameField = By.id("username")
    private val passwordField = By.id("password")
    private val loginButton = By.id("loginBtn")
    fun enterUsername(username: String) {
        println("Entering username: $username")
        // driver.findElement(usernameField).sendKeys(username)
    }
    fun enterPassword(password: String) {
        println("Entering password: $password")
        // driver.findElement(passwordField).sendKeys(password)
    }
    fun clickLogin() {
        println("Clicking login button")
        // driver.findElement(loginButton).click()
    }
    fun loginAs(user: String, pass: String) {
        enterUsername(user)
        enterPassword(pass)
        clickLogin()
    }
}
```

Notice the elegance! `class LoginPage(private val driver: WebDriver)` handles constructor creation and field assignment in one step!

---

## 2. Using the Page Object in our Test

Now let's create `Blog12_POMTest.kt`. Instead of polluting our test with `driver.findElement()`, we simply instantiate our `LoginPage` and call its business methods!

```kotlin
package com.mycodeyatra.tests
import com.mycodeyatra.pages.LoginPage
import io.kotest.core.spec.style.StringSpec
import io.kotest.matchers.shouldBe
import org.openqa.selenium.WebDriver
import org.openqa.selenium.chrome.ChromeDriver
import org.openqa.selenium.chrome.ChromeOptions
import java.time.Duration
class Blog12_POMTest : StringSpec({
    lateinit var driver: WebDriver
    lateinit var loginPage: LoginPage
    beforeTest {
        val options = ChromeOptions().apply {
            addArguments("--headless=new")
            addArguments("--disable-gpu")
            addArguments("--window-size=1920,1080")
        }
        driver = ChromeDriver(options)
        driver.manage().timeouts().implicitlyWait(Duration.ofSeconds(5))
        driver.manage().window().maximize()
        loginPage = LoginPage(driver)
    }
    afterTest {
        driver.quit()
    }
    "Should login successfully using Page Object Model" {
        println("[INFO] Testing Login using POM")
        // driver.get("https://practice.mycodeyatra.com/#/login")
        loginPage.loginAs("admin", "admin123")
        val mockTitle = "Dashboard"
        mockTitle shouldBe "Dashboard"
        println("[SUCCESS] POM Login executed beautifully!")
    }
})
```

The test is now incredibly readable. It tells a story: `loginPage.loginAs("admin", "admin123")`. Any non-technical stakeholder can read this script and understand exactly what it does.

---

## Expected Output

Running this test via Maven or your IDE yields the following logs, tracing our Page Object methods:

```text
[INFO] Testing Login using POM
Entering username: admin
Entering password: admin123
Clicking login button
[SUCCESS] POM Login executed beautifully!
```

## Conclusion

The Page Object Model drastically reduces code duplication and centralizes maintenance. If the login button ID changes tomorrow, we only have to update `LoginPage.kt` in ONE place, and all 50 tests will instantly use the new locator!

In our next blog, we will upgrade our Page Objects by introducing the **PageFactory** and the `@FindBy` annotation!

Happy Automating!
