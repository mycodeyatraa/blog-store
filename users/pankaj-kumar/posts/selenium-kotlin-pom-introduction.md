---
title: "Introduction to the Page Object Model (POM) Design Pattern in Selenium Kotlin"
date: "2025-03-01"
description: "Transition from messy, unmaintainable scripts to scalable architecture. Learn how to implement the Page Object Model (POM) design pattern in Selenium Kotlin."
tags: ["Selenium", "Kotlin", "POM", "Page Object Model", "Architecture", "Design Patterns"]
---

Up to this point in our series, we have been writing all of our Selenium logic—finding elements, typing text, clicking buttons, and asserting results—directly inside our test files. 

While this works for simple scripts, it becomes an absolute nightmare in large enterprise applications. If the ID of the login button changes, you would have to manually update dozens of test files! 

This is where the **Page Object Model (POM)** design pattern comes in. 

In this 24th post of our Selenium Kotlin Mastery Series, we will completely decouple our UI interactions from our test assertions to create a clean, maintainable, and scalable framework.

### What is the Page Object Model (POM)?

The Page Object Model dictates that every web page (or significant component of a page) should be represented by a dedicated Class in your code. 

1. **Locators** (e.g., `By.id("username")`) belong to the Page Class.
2. **Actions** (e.g., `clickLogin()`) belong to the Page Class.
3. **Assertions** (`shouldBe`) belong strictly to the Test Class.

The test class simply instantiates the Page Class and calls its action methods.

### Step 1: Creating the Page Class

Let's refactor the classic Login scenario on [The Internet](https://the-internet.herokuapp.com/login) into a Page Object. Notice how we use Kotlin's `by lazy` delegate. This ensures that the elements are only searched for *when they are actually needed*, rather than the moment the class is instantiated.

```kotlin
package com.mycodeyatra.pages
import com.mycodeyatra.utils.waitForElementVisible
import org.openqa.selenium.By
import org.openqa.selenium.WebDriver
import org.openqa.selenium.WebElement
class Blog24_LoginPage(private val driver: WebDriver) {
    // Locators defined using 'by lazy' for deferred evaluation
    private val usernameField: WebElement by lazy {
        driver.waitForElementVisible(By.id("username"))
    }
    private val passwordField: WebElement by lazy {
        driver.waitForElementVisible(By.id("password"))
    }
    private val loginButton: WebElement by lazy {
        driver.waitForElementVisible(By.cssSelector("button[type='submit']"))
    }
    private val flashMessage: WebElement by lazy {
        driver.waitForElementVisible(By.id("flash"))
    }
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
    fun getFlashMessageText(): String {
        return flashMessage.text
    }
}
```

By abstracting away the raw `driver.findElement` calls, this Page Class acts as an API for the Login Page. If the UI changes, you only update this one class.

### Step 2: Creating the Test Class

Now, let's write our Kotest script. Notice how clean and readable it is. It reads almost like plain English.

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
    "Login using Page Object Model (POM)" {
        val webDriver = driver ?: throw IllegalStateException("Driver not initialized")
        webDriver.get("https://the-internet.herokuapp.com/login")
        // Initialize the Page Object
        val loginPage = Blog24_LoginPage(webDriver)
        // Perform actions using the page object methods
        loginPage.enterUsername("tomsmith")
        loginPage.enterPassword("SuperSecretPassword!")
        loginPage.clickLogin()
        // Verify outcome using page object state
        val successMessage = loginPage.getFlashMessageText()
        successMessage shouldContain "You logged into a secure area!"
    }
})
```

### Execution Output

```
Initializing Headless Chrome Driver for CI/CD...
Waiting up to 10 seconds for element: By.id: username
Waiting up to 10 seconds for element: By.id: password
Waiting up to 10 seconds for element: By.cssSelector: button[type='submit']
Waiting up to 10 seconds for element: By.id: flash
Tests: 1, Passed: 1, Failed: 0
```

### Conclusion

The Page Object Model is not just a nice-to-have; it is an absolute necessity for any serious automation project. It enforces the **Separation of Concerns** principle, keeping your test logic entirely separate from your UI scraping logic.

In our next blog, we will take POM to the next level by introducing the **PageFactory** class and the `@FindBy` annotation!
