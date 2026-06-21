---
title: "Uploading and Downloading files using Selenium Kotlin"
date: "2025-02-24"
description: "Master file handling in Selenium Kotlin by learning how to seamlessly upload files using sendKeys and download files by configuring ChromeOptions."
tags: ["Selenium", "Kotlin", "File Handling", "Automation", "Kotest"]
---

Handling files is a critical aspect of test automation. Whether it is uploading an image to a user profile or downloading an invoice report, you need a robust way to interact with the file system. In this 19th post of our Selenium Kotlin Mastery Series, we will explore exactly how to accomplish both tasks.

We will focus on the most reliable methods: using `sendKeys` for file uploads, and configuring `ChromeOptions` for managing downloads effectively in headless or CI environments. 

Let's dive in!

### The Challenge with File Dialogs

Native OS file picker dialogs cannot be automated directly with Selenium because Selenium only interacts with the web browser DOM. 

Fortunately, HTML provides a `<input type="file">` element. By directly sending the absolute path of the file to this element, we can bypass the OS dialog completely and upload our file instantly. 

For downloading, browser behavior normally prompts the user for a save location. We can bypass this prompt by instructing the browser to download files automatically to a specified directory via `ChromeOptions`.

### Step 1: Upgrading our DriverManager for Downloads

To handle file downloads, we need to instruct our ChromeDriver where to save files without asking the user. We will update our `DriverManager.kt` to accept an optional download directory.

```kotlin
package com.mycodeyatra.utils

import org.openqa.selenium.WebDriver
import org.openqa.selenium.chrome.ChromeDriver
import org.openqa.selenium.chrome.ChromeOptions

object DriverManager {

    fun getHeadlessChromeDriver(downloadDir: String? = null): WebDriver {
        println("Initializing Headless Chrome Driver for CI/CD...")
        
        val options = ChromeOptions().apply {
            addArguments("--headless=new")
            addArguments("--no-sandbox")
            addArguments("--disable-dev-shm-usage")
            addArguments("--window-size=1920,1080")
            addArguments("--disable-blink-features=AutomationControlled")
            setExperimentalOption("excludeSwitches", arrayOf("enable-automation"))
            
            // Allow setting a custom download directory if provided
            downloadDir?.let {
                val prefs = mapOf(
                    "download.default_directory" to it,
                    "download.prompt_for_download" to false,
                    "download.directory_upgrade" to true,
                    "safebrowsing.enabled" to true
                )
                setExperimentalOption("prefs", prefs)
            }
        }
        
        return ChromeDriver(options)
    }
}
```

### Step 2: Writing the Upload and Download Tests

Now, let's write our Kotest specs. We will create a temporary directory before our tests start, pass it to our `DriverManager`, and then verify that files are successfully downloaded to it. For uploads, we will create a temporary file and upload it.

```kotlin
package com.mycodeyatra.tests

import com.mycodeyatra.utils.DriverManager
import com.mycodeyatra.utils.waitForElementVisible
import io.kotest.core.spec.style.StringSpec
import io.kotest.matchers.shouldBe
import io.kotest.matchers.string.shouldContain
import org.openqa.selenium.By
import org.openqa.selenium.WebDriver
import java.io.File
import kotlin.io.path.createTempDirectory

class Blog19_FileHandlingTest : StringSpec({

    var driver: WebDriver? = null
    lateinit var downloadDir: File

    beforeSpec {
        // Create a temporary directory for downloads
        downloadDir = createTempDirectory("selenium-downloads").toFile()
        // Initialize WebDriver with the custom download directory
        driver = DriverManager.getHeadlessChromeDriver(downloadDir = downloadDir.absolutePath)
        driver?.manage()?.window()?.maximize()
    }

    afterSpec {
        driver?.quit()
        // Clean up downloaded files and directory after test
        downloadDir.deleteRecursively()
    }

    "Upload a file successfully using sendKeys" {
        val webDriver = driver ?: throw IllegalStateException("Driver not initialized")
        
        webDriver.get("https://the-internet.herokuapp.com/upload")
        
        // Create a dummy file to upload
        val tempFileToUpload = File.createTempFile("test-upload", ".txt")
        tempFileToUpload.writeText("This is a test file for upload.")
        
        // Find the file input element and send the absolute path of the file
        val fileUploadInput = webDriver.findElement(By.id("file-upload"))
        fileUploadInput.sendKeys(tempFileToUpload.absolutePath)
        
        // Click the upload button
        webDriver.findElement(By.id("file-submit")).click()
        
        // Verify successful upload
        val uploadedFiles = webDriver.waitForElementVisible(By.id("uploaded-files"), 10)
        uploadedFiles.text shouldContain tempFileToUpload.name
        
        // Clean up the dummy file
        tempFileToUpload.delete()
    }

    "Download a file and verify its presence" {
        val webDriver = driver ?: throw IllegalStateException("Driver not initialized")
        
        webDriver.get("https://the-internet.herokuapp.com/download")
        
        // Find a file link to download (first available link)
        val downloadLinks = webDriver.findElements(By.cssSelector(".example a"))
        if (downloadLinks.isNotEmpty()) {
            val fileLink = downloadLinks.first()
            val fileName = fileLink.text
            
            // Click to download
            fileLink.click()
            
            // Wait for the file to be downloaded by polling the download directory
            var fileDownloaded = false
            for (i in 1..10) {
                val files = downloadDir.listFiles()
                if (files != null && files.any { it.name == fileName }) {
                    fileDownloaded = true
                    println("File downloaded successfully: $fileName")
                    break
                }
                Thread.sleep(1000) // Wait 1 second before checking again
            }
            
            fileDownloaded shouldBe true
        } else {
            println("No files available to download on the page.")
        }
    }
})
```

### Explanation of the Approach

1. **Upload Approach:** We avoid interacting with complex OS-level scripts (like AutoIt or Robot class) which are brittle. By creating a temporary text file, resolving its `absolutePath`, and executing `.sendKeys()` on the `<input type="file">`, the browser handles the upload natively.
2. **Download Setup:** By modifying the experimental `prefs` inside `ChromeOptions`, we specify `"download.default_directory"` to point to our isolated test directory. `"download.prompt_for_download" to false` ensures the dialog box doesn't hang our execution.
3. **Download Verification:** Downloading files takes time depending on network speed. A simple polling loop checks our target directory continuously until the file is found or a timeout is reached. 
4. **Cleanup:** Good tests clean up after themselves. `deleteRecursively()` in `afterSpec` ensures your machine isn't cluttered with temporary files.

### Execution Output

```
Initializing Headless Chrome Driver for CI/CD...
Waiting up to 10 seconds for element: By.id: uploaded-files
File downloaded successfully: some-file.txt

Tests: 2, Passed: 2, Failed: 0
```

### Conclusion

Handling files with Selenium Kotlin doesn't have to be a headache. By leveraging native DOM injection (`sendKeys`) for uploads and browser-level configurations (`ChromeOptions`) for downloads, we maintain a pure, cross-platform automation solution that works seamlessly locally and inside CI pipelines. 

In our next blog, we will tackle taking Screenshots during our tests, ensuring we capture crucial evidence when tests fail. Happy automating!
