---
title: "Visual Regression Testing in Selenium Kotlin using AShot"
date: "2025-03-08"
description: "Detect pixel-level UI changes in your web application by integrating AShot for Visual Regression Testing directly inside your Kotlin automation framework."
tags: ["Selenium", "Kotlin", "AShot", "Visual Testing", "Regression", "UI"]
---

Welcome to the 31st post of our Selenium Kotlin Mastery Series!

Traditional Selenium scripts are excellent at verifying **functionality**—ensuring that a button clicks or a text field accepts input. But what happens if a CSS change turns your login button invisible, or shifts the entire form off the screen? Selenium will still successfully "click" the DOM element, but your users will see a broken layout.

To detect UI styling breaks, we need **Visual Regression Testing**. In this post, we will integrate **AShot**, a powerful image comparison library, to perform pixel-perfect image validation in Kotlin.

### Step 1: Adding AShot Dependency

Open your `pom.xml` and add the AShot library:

```xml
<dependency>
    <groupId>ru.yandex.qatools.ashot</groupId>
    <artifactId>ashot</artifactId>
    <version>1.5.4</version>
</dependency>
```

### Step 2: The Visual Testing Logic

Visual testing generally follows a three-step process:
1. **Baseline Creation**: The first time the test runs, it takes a screenshot and saves it as the "expected" baseline.
2. **Actual Capture**: On subsequent runs, it captures a new "actual" screenshot.
3. **Comparison**: It compares the actual image to the baseline. If they differ, the test fails and generates a "diff" image highlighting the mismatches.

### Step 3: Implementing AShot in Kotest

Here is how we capture and compare specific WebElements (like the login container) dynamically.

```kotlin
package com.mycodeyatra.tests
import com.mycodeyatra.utils.ThreadSafeDriverManager
import io.kotest.core.spec.style.StringSpec
import io.kotest.matchers.shouldBe
import org.openqa.selenium.By
import org.openqa.selenium.support.ui.ExpectedConditions
import org.openqa.selenium.support.ui.WebDriverWait
import ru.yandex.qatools.ashot.AShot
import ru.yandex.qatools.ashot.comparison.ImageDiff
import ru.yandex.qatools.ashot.comparison.ImageDiffer
import ru.yandex.qatools.ashot.coordinates.WebDriverCoordsProvider
import java.io.File
import javax.imageio.ImageIO
class Blog31_VisualTestingTest : StringSpec({
    afterSpec {
        ThreadSafeDriverManager.quitDriver()
    }
    "Perform Visual Regression on Login Container" {
        val driver = ThreadSafeDriverManager.getDriver()
        driver.get("https://practice.mycodeyatra.com/#/login")
        // Wait for container using explicit waits
        val wait = WebDriverWait(driver, java.time.Duration.ofSeconds(10))
        val loginContainer = wait.until(ExpectedConditions.visibilityOfElementLocated(By.tagName("form")))
        // Take Screenshot of the Element using AShot
        val actualScreenshot = AShot()
            .coordsProvider(WebDriverCoordsProvider())
            .takeScreenshot(driver, loginContainer)
        val baselineFile = File("screenshots/login_baseline.png")
        if (!baselineFile.exists()) {
            // First run: Save as baseline
            baselineFile.parentFile.mkdirs()
            ImageIO.write(actualScreenshot.image, "png", baselineFile)
            println("Baseline image created at ${baselineFile.absolutePath}")
        } else {
            // Subsequent runs: Compare actual with baseline
            val expectedImage = ImageIO.read(baselineFile)
            val imageDiffer = ImageDiffer()
            val diff: ImageDiff = imageDiffer.makeDiff(expectedImage, actualScreenshot.image)
            if (diff.hasDiff()) {
                val diffFile = File("screenshots/login_diff.png")
                ImageIO.write(diff.markedImage, "png", diffFile)
                println("Visual difference found! Diff image saved at ${diffFile.absolutePath}")
            }
            diff.hasDiff() shouldBe false
        }
    }
})
```

### How AShot Works Under the Hood

1. **`coordsProvider(WebDriverCoordsProvider())`**: AShot uses coordinate providers to precisely map the `WebElement` coordinates to the screenshot pixels. This avoids issues with different browser viewport sizes.
2. **`ImageDiffer`**: This class does the heavy pixel-by-pixel mathematical comparison.
3. **`diff.markedImage`**: If a difference is found, AShot automatically generates a composite image highlighting the differing pixels in bright red, saving you hours of manual debugging.

### Conclusion

By layering Visual Regression Testing on top of your functional automation, you achieve true end-to-end coverage that guarantees both functionality and aesthetics!

In our next blog, we will cover **Dockerizing Selenium Kotlin**, showing how to run our tests inside scalable containers!
