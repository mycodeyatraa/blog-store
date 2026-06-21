---
title: "Introduction to the Page Object Model (POM) Design Pattern in Selenium Kotlin"
date: "2025-03-01"
description: "Transition from messy, unmaintainable scripts to scalable architecture. Learn how to implement the Page Object Model (POM) design pattern in Selenium Kotlin using our live practice sandbox."
tags: ["Selenium", "Kotlin", "POM", "Page Object Model", "Architecture", "Design Patterns"]
---

Up to this point in our series, we have been writing all of our Selenium logic—finding elements, typing text, clicking buttons, and asserting results—directly inside our test files. 

While this works for simple scripts, it becomes an absolute nightmare in large enterprise applications. If the `data-testid` of the login button changes, you would have to manually update dozens of test files! 

This is where the **Page Object Model (POM)** design pattern comes in. 

*Note: In older Selenium versions (especially in Java), developers relied heavily on the `PageFactory` class and `@FindBy` annotations. However, PageFactory is largely considered **deprecated** and clunky in modern automation. In Kotlin, we use native language features like `by lazy` for a cleaner, fluent approach.*

In this 24th post of our Selenium Kotlin Mastery Series, we will decouple our UI interactions from our test assertions to create a clean framework using our live **MyCodeYatra Practice Sandbox**.

### What is the Page Object Model (POM)?

The Page Object Model dictates that every web page (or significant component of a page) should be represented by a dedicated Class in your code. 

1. **Locators** (e.g., `By.cssSelector("[data-testid='username']")`) belong to the Page Class.
2. **Actions** (e.g., `clickLogin()`) belong to the Page Class.
3. **Assertions** (`shouldBe`) belong strictly to the Test Class.

### Step 1: Creating the Page Class

Let's automate the login flow on our official test site: [Practice MyCodeYatra Login](https://practice.mycodeyatra.com/#/login). 

Notice how we use Kotlin's custom getters (`get()`). This ensures that elements are dynamically searched for *only when they are interacted with*, and uniquely *every time* they are interacted with. This completely prevents `StaleElementReferenceException` issues on page reloads, acting exactly like `PageFactory`'s dynamic proxies but without the bulky annotations!

```kotlin
package com.mycodeyatra.pages
import com.mycodeyatra.utils.waitForElementVisible
import org.openqa.selenium.By
import org.openqa.selenium.WebDriver
import org.openqa.selenium.WebElement
class Blog24_LoginPage(private val driver: WebDriver) {
    // Locators defined using custom getters for dynamic evaluation
    private val usernameField: WebElement get() = driver.waitForElementVisible(By.cssSelector("[data-testid='username']"))
    private val passwordField: WebElement get() = driver.waitForElementVisible(By.cssSelector("[data-testid='password']"))
    private val loginButton: WebElement get() = driver.waitForElementVisible(By.cssSelector("[data-testid='login-btn']"))
    private val profileTitle: WebElement get() = driver.waitForElementVisible(By.cssSelector("[data-testid='profile-title']"))
    // Page Actions
    fun enterUsername(user: String) {
        usernameField.clear()
        usernameField.sendKeys(user)
    }
    fun enterPassword(pass: String) {
        passwordField.clear()
        passwordField.sendKeys(pass)
    }
    fun clickLogin() {
        loginButton.click()
    }
    fun getProfileTitle(): String {
        return profileTitle.text
    }
}
```

By abstracting away the raw `driver.findElement` calls, this Page Class acts as an API for the Login Page. If the UI changes, you only update this one class.

### Step 2: Creating the Test Class

Now, let's write our Kotest script. The test logic simply instantiates the Page Object and interacts with it, leaving it completely agnostic of how the locators actually work underneath.

```kotlin
package com.mycodeyatra.tests
import com.mycodeyatra.pages.Blog24_LoginPage
import com.mycodeyatra.utils.DriverManager
import io.kotest.core.spec.style.StringSpec
import io.kotest.matchers.string.shouldContain
import org.openqa.selenium.WebDriver
class Blog24_POMTest : StringSpec({
    var driver: WebDriver? = null
    beforeSpec {
        driver = DriverManager.getHeadlessChromeDriver()
        driver?.manage()?.window()?.maximize()
    }
    afterSpec {
        driver?.quit()
    }
    "Login using Page Object Model (POM) on Live URL" {
        val webDriver = driver ?: throw IllegalStateException("Driver not initialized")
        webDriver.get("https://practice.mycodeyatra.com/#/login")
        // Initialize the Page Object
        val loginPage = Blog24_LoginPage(webDriver)
        // Perform actions using the page object methods
        loginPage.enterUsername("admin")
        loginPage.enterPassword("admin123")
        loginPage.clickLogin()
        // Verify outcome using page object state
        val successMessage = loginPage.getProfileTitle()
        successMessage shouldContain "Welcome back, admin"
    }
})
```

### Execution Output

```
Initializing Headless Chrome Driver for CI/CD...
Waiting up to 10 seconds for element: By.cssSelector: [data-testid='username']
Waiting up to 10 seconds for element: By.cssSelector: [data-testid='password']
Waiting up to 10 seconds for element: By.cssSelector: [data-testid='login-btn']
Waiting up to 10 seconds for element: By.cssSelector: [data-testid='profile-title']
Tests: 1, Passed: 1, Failed: 0
```

### Conclusion

The Page Object Model is not just a nice-to-have; it is an absolute necessity for any serious automation project. By relying on native Kotlin features like custom getters (`get()`) rather than deprecated Java annotations, we've built a robust, modern framework layer!

In our next blog, we will introduce **Data-Driven Testing in Selenium Kotlin**, showing you how to loop through multiple datasets gracefully!
