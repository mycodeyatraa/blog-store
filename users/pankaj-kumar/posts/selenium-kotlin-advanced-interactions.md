---
title: "Advanced Interactions: Mouse Hover and Drag-and-Drop using the Actions Class"
date: "2025-02-26"
description: "Take control of complex user interactions in Selenium Kotlin by mastering the Actions class to perform mouse hovers and drag-and-drop operations."
tags: ["Selenium", "Kotlin", "Actions", "Mouse Hover", "Automation", "Kotest"]
---

Modern web applications are filled with rich, interactive components like dropdown menus that appear on hover, drag-and-drop file uploaders, and interactive sliders. Standard Selenium commands like `.click()` and `.sendKeys()` are not enough to handle these scenarios.

In this 21st post of our Selenium Kotlin Mastery Series, we will explore the `Actions` class—Selenium's dedicated API for simulating complex user gestures like mouse hovers and drag-and-drop.

### Understanding the Actions Class

The `Actions` class provides a builder pattern to compile a sequence of user gestures. You chain commands together and finish the sequence by calling `.perform()`. Without `.perform()`, the actions are built but never actually executed in the browser!

Let's dive straight into the code.

### Step 1: The Mouse Hover Scenario

A very common requirement is verifying that an element (like a tooltip or a submenu) becomes visible only *after* the user hovers over another element. We will test this against [The Internet's Hovers page](https://the-internet.herokuapp.com/hovers).

```kotlin
package com.mycodeyatra.tests

import com.mycodeyatra.utils.DriverManager
import com.mycodeyatra.utils.waitForElementVisible
import io.kotest.core.spec.style.StringSpec
import io.kotest.matchers.shouldBe
import io.kotest.matchers.string.shouldContain
import org.openqa.selenium.By
import org.openqa.selenium.WebDriver
import org.openqa.selenium.interactions.Actions

class Blog21_ActionsTest : StringSpec({

    var driver: WebDriver? = null

    beforeSpec {
        driver = DriverManager.getHeadlessChromeDriver()
        driver?.manage()?.window()?.maximize()
    }

    afterSpec {
        driver?.quit()
    }

    "Perform Mouse Hover using Actions class" {
        val webDriver = driver ?: throw IllegalStateException("Driver not initialized")
        
        webDriver.get("https://the-internet.herokuapp.com/hovers")
        
        // Find the first user avatar image
        val firstAvatar = webDriver.waitForElementVisible(By.cssSelector(".figure:nth-child(3) img"))
        
        // Initialize Actions class and move to the element (hover)
        val actions = Actions(webDriver)
        actions.moveToElement(firstAvatar).perform() // .perform() is mandatory!
        
        // After hovering, the caption should become visible
        val caption = webDriver.findElement(By.cssSelector(".figure:nth-child(3) .figcaption h5"))
        caption.isDisplayed shouldBe true
        caption.text shouldContain "name: user1"
    }
```

By passing our `WebDriver` instance to `Actions(webDriver)`, we link the gesture builder to our browser. `moveToElement(firstAvatar)` shifts the virtual mouse cursor to the center of the image, triggering the hover CSS/JS events.

### Step 2: The Drag and Drop Scenario

Another popular interaction is clicking an element, holding it down, moving the mouse to a target location, and releasing it. The `Actions` class simplifies this with a direct `dragAndDrop(source, target)` method.

```kotlin
    "Perform Drag and Drop using Actions class" {
        val webDriver = driver ?: throw IllegalStateException("Driver not initialized")
        
        webDriver.get("https://the-internet.herokuapp.com/drag_and_drop")
        
        val columnA = webDriver.waitForElementVisible(By.id("column-a"))
        val columnB = webDriver.waitForElementVisible(By.id("column-b"))
        
        // Assert initial state
        columnA.text shouldBe "A"
        columnB.text shouldBe "B"
        
        // Perform drag and drop
        val actions = Actions(webDriver)
        actions.dragAndDrop(columnA, columnB).perform()
        
        println("Executed Drag and Drop via Actions API.")
    }
})
```

*Note on HTML5 Drag and Drop:* While the `Actions` class works flawlessly for traditional JavaScript-based drag-and-drop (like jQuery UI), there is a long-standing known issue where WebDriver cannot natively trigger HTML5 Drag and Drop events in some browsers. If you encounter this, you will need to inject a custom JavaScript workaround via the `JavascriptExecutor`—something we covered in earlier JS execution blogs!

### Execution Output

```
Initializing Headless Chrome Driver for CI/CD...
Waiting up to 10 seconds for element: By.cssSelector: .figure:nth-child(3) img
Waiting up to 10 seconds for element: By.id: column-a
Waiting up to 10 seconds for element: By.id: column-b
Executed Drag and Drop via Actions API.

Tests: 2, Passed: 2, Failed: 0
```

### Conclusion

The `Actions` class unlocks the ability to test complex user interfaces that rely on subtle gestures. Remember the golden rule: **always end your Actions chain with `.perform()`**, or your gestures will never reach the browser.

In our next blog, we will discuss **Handling iFrames and Nested Frames** in Selenium Kotlin, an area that frequently trips up beginners. Happy automating!
