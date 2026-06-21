---
title: "Screenplay Pattern in Selenium Kotlin: Refactoring POM with SOLID Principles"
date: "2025-03-05"
description: "Elevate your automation architecture by implementing a lightweight Screenplay Pattern in Kotlin, focusing on Actions, Tasks, and Actors rather than bare Page Objects."
tags: ["Selenium", "Kotlin", "Screenplay", "Design Patterns", "SOLID", "Architecture"]
---

The **Page Object Model (POM)** is the undisputed king of Selenium design patterns. However, as test suites grow to thousands of scripts, traditional POM tends to violate the **Single Responsibility Principle** (the 'S' in SOLID). Page Objects become massive "God Classes" bloated with hundreds of methods for every conceivable user action.

Enter the **Screenplay Pattern**. Originally popularized by the Serenity BDD framework, Screenplay shifts the focus from "Pages" to "Actors" and "Tasks". 

In this 5th post of our **Framework Design & Patterns** series, we will implement a lightweight, native Kotlin Screenplay architecture for our [MyCodeYatra Practice Site](https://practice.mycodeyatra.com/#/sandbox).

### The Screenplay Vocabulary

Instead of saying *`loginPage.enterCredentials()`*, Screenplay forces you to write sentences:
**`Pankaj` (the Actor) `attemptsTo` (the Action) `LoginWithCredentials` (the Task).**

Let's build this DSL in Kotlin!

### Step 1: The Actor and Task Interfaces

First, we define our `Actor`. An Actor holds a reference to the WebDriver and has the ability to perform a list of `Tasks`.

```kotlin
package com.mycodeyatra.screenplay
import org.openqa.selenium.WebDriver
// The Actor is the entity performing the actions (e.g., "Admin", "User")
class Actor(val name: String, val driver: WebDriver) {
    fun attemptsTo(vararg tasks: Task) {
        for (task in tasks) {
            task.performAs(this)
        }
    }
}
// A Task represents a high-level business action
interface Task {
    fun performAs(actor: Actor)
}
```

### Step 2: Creating a Concrete Task

A Task encapsulates a sequence of steps. Notice how the `LoginTask` strictly handles the *behavior* of logging in, abstracting it away from the Test class. Under the hood, it still delegates the locators to our `Blog24_LoginPage` object.

```kotlin
package com.mycodeyatra.screenplay.tasks
import com.mycodeyatra.models.TestUser
import com.mycodeyatra.pages.Blog24_LoginPage
import com.mycodeyatra.screenplay.Actor
import com.mycodeyatra.screenplay.Task
class LoginTask(private val user: TestUser) : Task {
    override fun performAs(actor: Actor) {
        // Navigate
        actor.driver.get("https://practice.mycodeyatra.com/#/login")
        // Interact using the Page Object under the hood
        val loginPage = Blog24_LoginPage(actor.driver)
        loginPage.enterUsername(user.username)
        loginPage.enterPassword(user.password)
        loginPage.clickLogin()
    }
    companion object {
        fun withCredentials(user: TestUser): LoginTask {
            return LoginTask(user)
        }
    }
}
```

### Step 3: Writing the Screenplay Test

Let's see how beautiful and readable our Kotest spec becomes when we string this together.

```kotlin
package com.mycodeyatra.tests
import com.mycodeyatra.models.TestUser
import com.mycodeyatra.pages.Blog24_LoginPage
import com.mycodeyatra.screenplay.Actor
import com.mycodeyatra.screenplay.tasks.LoginTask
import com.mycodeyatra.utils.ThreadSafeDriverManager
import io.kotest.core.spec.style.StringSpec
import io.kotest.matchers.string.shouldContain
class Blog28_ScreenplayTest : StringSpec({
    afterSpec {
        ThreadSafeDriverManager.quitDriver()
    }
    "Test Login using the Screenplay Pattern" {
        val webDriver = ThreadSafeDriverManager.getDriver()
        // 1. Define the Actor
        val pankaj = Actor("Pankaj", webDriver)
        // 2. Define the Test Data
        val validUser = TestUser(username = "admin", password = "admin123")
        // 3. The Actor attempts a Task
        pankaj.attemptsTo(
            LoginTask.withCredentials(validUser)
        )
        // 4. Assert the Outcome
        val loginPage = Blog24_LoginPage(webDriver)
        loginPage.getProfileTitle() shouldContain "Welcome back, admin"
    }
})
```

### Execution Output

```
[INFO] Running com.mycodeyatra.tests.Blog28_ScreenplayTest
Initializing new Headless Chrome Driver for Thread: 1
Quitting Driver for Thread: 1
Tests: 1, Passed: 1, Failed: 0
```

### Conclusion

The Screenplay Pattern brings ultimate clarity to automation code. By treating users as `Actors` performing `Tasks`, the code reads like a plain English business requirement. This pattern is infinitely scalable and prevents your Page Objects from becoming bloated nightmares.

In our next blog, we will transition into Phase 4, diving into **Advanced Scenarios** starting with **File Downloads and Uploads in Selenium Kotlin**!
