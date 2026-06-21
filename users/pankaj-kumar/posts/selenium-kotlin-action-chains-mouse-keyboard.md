---
title: "Advanced Interactions using Action Chains in Selenium Kotlin"
date: "21-Jun-2026"
description: "Master complex interactions in Selenium Kotlin using the Actions class. Learn how to perform Mouse Hovers, Double Clicks, Right Clicks, and Keyboard Chaining."
categories: ["Selenium Kotlin", "Test Automation"]
tags: ["Selenium", "Kotlin", "Kotest", "Actions", "MouseHover", "RightClick", "Keyboard"]
author: "Pankaj Kumar"
lastUpdated: "21-Jun-2026"
---

Welcome back to the **Selenium Kotlin Mastery Series**!

We have already covered basic interactions like `.click()` and `.sendKeys()`. But what happens when you need to automate complex user behaviors? How do you hover over a mega-menu to reveal dropdown links? How do you double-click a file, or right-click an image?

Enter the **`Actions`** class—Selenium's dedicated API for advanced Mouse and Keyboard interactions!

---

## 1. The Anatomy of the Actions Class

The `Actions` class resides in the `org.openqa.selenium.interactions.Actions` package. 

When using `Actions`, you define a sequence of behaviors (chaining) and then trigger them all at once using the `.perform()` method. If you forget to add `.perform()` at the end of your chain, **nothing will happen**!

### Core Mouse & Keyboard Methods

Here are the most common methods you will use:
1. **Hovering**: `moveToElement(element)`
2. **Double Click**: `doubleClick(element)`
3. **Right Click**: `contextClick(element)`
4. **Drag and Drop**: `dragAndDrop(source, target)`
5. **Keyboard Presses**: `keyDown(Keys.SHIFT)` and `keyUp(Keys.SHIFT)`

---

## 2. Writing the Kotlin Script

Let's put this into practice! We will create a new file called `Blog9_ActionChainsTest.kt` in `src/test/kotlin/com/mycodeyatra/tests/`. 

In this script, we will hover over a button, double-click another button, right-click a header, and finally use a Keyboard Chain to hold `SHIFT` while typing a name!

```kotlin
package com.mycodeyatra.tests

import io.kotest.core.spec.style.StringSpec
import io.kotest.matchers.string.shouldContain
import org.openqa.selenium.By
import org.openqa.selenium.Keys
import org.openqa.selenium.WebDriver
import org.openqa.selenium.chrome.ChromeDriver
import org.openqa.selenium.chrome.ChromeOptions
import org.openqa.selenium.interactions.Actions
import java.time.Duration

class Blog9_ActionChainsTest : StringSpec({
    lateinit var driver: WebDriver
    lateinit var actions: Actions

    beforeTest {
        val options = ChromeOptions().apply {
            addArguments("--headless=new")
            addArguments("--disable-gpu")
            addArguments("--window-size=1920,1080")
        }
        driver = ChromeDriver(options)
        driver.manage().timeouts().implicitlyWait(Duration.ofSeconds(5))
        driver.manage().window().maximize()
        
        // Instantiate the Actions class
        actions = Actions(driver)
    }

    afterTest {
        driver.quit()
    }

    "Should perform complex mouse and keyboard interactions using Actions class" {
        println("[INFO] Running com.mycodeyatra.tests.Blog9_ActionChainsTest")
        driver.get("https://practice.mycodeyatra.com/#/form-practice")
        println("Navigated to Form Practice Page.")

        // 1. MOUSE HOVER
        println("\n--- 1. MOUSE HOVER (moveToElement) ---")
        val submitBtn = driver.findElement(By.xpath("//button[text()='Submit Form']"))
        // Perform a hover over the Submit button
        actions.moveToElement(submitBtn).perform()
        println("Successfully hovered over the Submit Button.")

        // 2. DOUBLE CLICK
        println("\n--- 2. DOUBLE CLICK ---")
        val clearBtn = driver.findElement(By.xpath("//button[text()='Clear']"))
        actions.doubleClick(clearBtn).perform()
        println("Successfully double-clicked the Clear Button.")

        // 3. RIGHT CLICK (Context Click)
        println("\n--- 3. RIGHT CLICK (contextClick) ---")
        val header = driver.findElement(By.tagName("h2"))
        actions.contextClick(header).perform()
        println("Successfully right-clicked the Form Header.")

        // 4. KEYBOARD ACTIONS
        println("\n--- 4. KEYBOARD ACTIONS ---")
        val fullNameInput = driver.findElement(By.name("fullName"))
        
        // Click into the input, hold SHIFT, type text (it will be uppercase), release SHIFT
        actions.click(fullNameInput)
            .keyDown(Keys.SHIFT)
            .sendKeys("actions class rocks")
            .keyUp(Keys.SHIFT)
            .perform()
            
        val enteredText = fullNameInput.getAttribute("value")
        println("Typed text using Keyboard Shift: $enteredText")
        
        enteredText shouldContain "ACTIONS CLASS ROCKS"
        println("\nActions script executed successfully!")
    }
})
```

> **💡 Action Chaining:** Notice in the Keyboard section how we strung multiple methods together: `.click().keyDown().sendKeys().keyUp().perform()`. This builds a composite action sequence that Selenium executes rapidly in order!

---

## Expected Output

When executing this test suite via Kotest, you will see the following output in your IntelliJ/Maven console confirming that all actions were executed correctly:

```text
[INFO] Running com.mycodeyatra.tests.Blog9_ActionChainsTest
Navigated to Form Practice Page.

--- 1. MOUSE HOVER (moveToElement) ---
Successfully hovered over the Submit Button.

--- 2. DOUBLE CLICK ---
Successfully double-clicked the Clear Button.

--- 3. RIGHT CLICK (contextClick) ---
Successfully right-clicked the Form Header.

--- 4. KEYBOARD ACTIONS ---
Typed text using Keyboard Shift: ACTIONS CLASS ROCKS

Actions script executed successfully!
```

## Conclusion

You now have complete mastery over the user's physical hardware! The `Actions` class bridges the gap between simple test scripts and hyper-realistic user simulations.

In our next blog, we will take a deeper dive into **Handling Dynamic Data** and Data-Driven Testing (DDT) using Kotlin's powerful data classes!

Happy Automating!
