---
title: "File Uploads and Downloads in Kotlin"
date: "12-Feb-2025"
description: "Learn how to bypass native OS dialogs to seamlessly upload files and manage automated downloads using Kotlin and Selenium."
categories: ["Selenium Kotlin", "Test Automation", "Files"]
tags: ["Selenium", "Kotlin", "Kotest", "Uploads", "Downloads"]
author: "Pankaj Kumar"
lastUpdated: "12-Feb-2025"
---

Welcome back to the **Selenium Kotlin Mastery** series!

Handling files in browser automation can be tricky. When you click an "Upload" button, the browser opens a native OS file picker window (Windows Explorer or macOS Finder). **Selenium cannot interact with native OS windows!**

Fortunately, there is a simple workaround. Almost all file upload buttons are just HTML `<input type="file">` elements disguised with CSS. Today, we'll learn how to upload files by sending keystrokes directly to the input, and how to configure Chrome to download files silently!

---

## 1. File Uploads in Kotlin

Instead of clicking the upload button, we locate the underlying `input` element and send the absolute path of the file we want to upload.

Create `Blog7_FileUploadDownloadTest.kt` in your tests folder:
```kotlin
package com.mycodeyatra.tests
import io.kotest.core.spec.style.StringSpec
import io.kotest.matchers.shouldBe
import org.openqa.selenium.By
import org.openqa.selenium.WebDriver
import org.openqa.selenium.chrome.ChromeDriver
import java.io.File
class Blog7_FileUploadDownloadTest : StringSpec({
    lateinit var driver: WebDriver
    beforeTest {
        driver = ChromeDriver()
        driver.manage().window().maximize()
    }
    afterTest {
        driver.quit()
    }
    "Should upload a file seamlessly" {
        driver.get("https://mycodeyatra.com/practice/upload")
        // 1. Create a dummy file locally
        val testFile = File.createTempFile("test_upload", ".txt")
        testFile.writeText("Hello from Kotlin Automation!")
        // 2. Locate the hidden <input type='file'> element
        val fileInput = driver.findElement(By.id("file-upload"))
        // 3. Send the ABSOLUTE path to the input
        fileInput.sendKeys(testFile.absolutePath)
        // 4. Click Submit
        driver.findElement(By.id("file-submit")).click()
        // 5. Verify successful upload
        val uploadedHeader = driver.findElement(By.tagName("h3")).text
        uploadedHeader shouldBe "File Uploaded!"
        // Clean up
        testFile.delete()
    }
```

---

## 2. Managing File Downloads

To handle downloads cleanly, we don't want Chrome to ask the user "Where do you want to save this file?". We want to inject **ChromeOptions** to force silent downloads into a specific directory.
```kotlin
    "Should download a file silently to a specific directory" {
        // 1. Define download directory
        val downloadDir = File(System.getProperty("user.dir"), "downloads").absolutePath
        File(downloadDir).mkdirs()
        // 2. Configure Chrome to download silently
        val prefs = mapOf(
            "download.default_directory" to downloadDir,
            "download.prompt_for_download" to false,
            "safebrowsing.enabled" to true
        )
        val options = org.openqa.selenium.chrome.ChromeOptions()
        options.setExperimentalOption("prefs", prefs)
        // Initialize driver with options
        val customDriver = ChromeDriver(options)
        try {
            customDriver.get("https://mycodeyatra.com/practice/download")
            // 3. Click download link
            customDriver.findElement(By.id("download-link")).click()
            // 4. The Kotlin Way: Wait for the file to appear using a tight loop and coroutines (or Thread.sleep for simplicity here)
            var fileExists = false
            for (i in 1..10) {
                val downloadedFile = File(downloadDir, "sample.pdf")
                if (downloadedFile.exists() && downloadedFile.length() > 0) {
                    fileExists = true
                    break
                }
                Thread.sleep(1000)
            }
            // 5. Assert it downloaded!
            fileExists shouldBe true
        } finally {
            customDriver.quit()
        }
    }
})
```

---

## 3. Expected Test Output

When executing this test suite via Kotest, you will see the following output in your IntelliJ/Maven console confirming that both the upload simulation and the download directory intercept worked perfectly:

```text
[INFO] Running com.mycodeyatra.tests.Blog7_FileUploadDownloadTest
Blog7_FileUploadDownloadTest
[PASS] Should upload a file seamlessly
[PASS] Should download a file silently to a specific directory
2 tests completed, 2 successes, 0 failures, 0 ignored.
```

---

## Conclusion

By understanding that OS dialogs are off-limits, we bypassed the UI completely. We sent keys directly to the DOM for uploads, and manipulated browser preferences for downloads.

In the next blog, we will master **Taking Element-Level and Full-Page Screenshots**, adding vital evidence gathering to our framework!

Happy Automating!
