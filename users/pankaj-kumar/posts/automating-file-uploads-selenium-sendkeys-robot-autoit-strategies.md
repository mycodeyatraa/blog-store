---
title: Automating File Uploads in Selenium: SendKeys, Robot, and AutoIt Strategies
date: 14-Jul-2024
lastUpdated: 11-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [selenium, java, file-upload, sendkeys, robot-class, autoit]
category: UI Automation
categories: [UI Automation, Selenium, Java]
excerpt: >-
  Master file upload automation in Selenium WebDriver. Learn how to upload files using the standard sendKeys strategy, Java Robot class, and AutoIt scripts in Java.
readTime: 7 min read
---

# Automating File Uploads in Selenium: SendKeys, Robot, and AutoIt Strategies

> 📅 **Last Updated:** 11-Jun-2026

File upload is a standard feature in modern web applications. Yet, automating uploads remains a common pain point. Why? Because clicking an "Upload" button launches a native OS file dialog window (Windows Explorer, macOS Finder, or Linux File Manager). 

Since Selenium WebDriver is strictly limited to browser automation, it cannot interact with operating system dialogues directly. In this tutorial, we will explore three strategies to automate file uploads in Java: **SendKeys (The standard strategy)**, **Java Robot Class**, and **AutoIt integration**.

---

## 🧭 File Upload Strategy Decision Engine

The flowchart below outlines how to choose the most robust file upload strategy based on the structure of the DOM and the runtime environment:

![diagram_1](https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/posts/automating-file-uploads-selenium-sendkeys-robot-autoit-strategies/images/diagram_1.png)

---

## 🛠️ Step-by-Step Code Walkthrough

The most robust strategy is using Selenium's standard `sendKeys` option. Instead of clicking the trigger container (which blocks tests by opening an OS window), we send the **absolute path** of the file directly to the hidden or visible `<input type="file">` tag.

Here is the complete implementation of this E2E file upload suite:

```java
package com.mycodeyatra.tests;
import io.github.bonigarcia.wdm.WebDriverManager;
import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.chrome.ChromeOptions;
import org.openqa.selenium.support.ui.ExpectedConditions;
import org.openqa.selenium.support.ui.WebDriverWait;
import org.testng.Assert;
import org.testng.annotations.AfterMethod;
import org.testng.annotations.BeforeMethod;
import org.testng.annotations.Test;
import java.io.File;
import java.io.FileWriter;
import java.io.IOException;
import java.time.Duration;
public class UploadTest {
    private WebDriver driver;
    private WebDriverWait wait;
    private File tempHtmlFile;
    private File uploadTxtFile;
    @BeforeMethod
    public void setUp() throws IOException {
        // Create local HTML form for file upload testing
        File targetDir = new File("target");
        if (!targetDir.exists()) {
            targetDir.mkdirs();
        }
        tempHtmlFile = new File(targetDir, "file_upload_demo.html");
        try (FileWriter writer = new FileWriter(tempHtmlFile)) {
            writer.write("<!DOCTYPE html>\n" +
                    "<html>\n" +
                    "<head>\n" +
                    "    <title>File Upload Practice Sandbox</title>\n" +
                    "</head>\n" +
                    "<body>\n" +
                    "    <h2>File Upload Demonstration</h2>\n" +
                    "    <p>Upload a file to test sendKeys automation strategy:</p>\n" +
                    "    <input type='file' id='file-upload' data-testid='file-upload-input' />\n" +
                    "    <div id='upload-status' data-testid='upload-status' style='margin-top:20px; font-weight:bold;'>\n" +
                    "        No file uploaded\n" +
                    "    </div>\n" +
                    "    <script>\n" +
                    "        document.getElementById('file-upload').addEventListener('change', function(e) {\n" +
                    "            var fileName = e.target.files[0] ? e.target.files[0].name : 'No file uploaded';\n" +
                    "            document.getElementById('upload-status').textContent = 'Successfully uploaded: ' + fileName;\n" +
                    "        });\n" +
                    "    </script>\n" +
                    "</body>\n" +
                    "</html>");
        }
        // Create dummy text file to upload
        uploadTxtFile = new File(targetDir, "upload_test_file.txt");
        try (FileWriter writer = new FileWriter(uploadTxtFile)) {
            writer.write("This is dummy text content for file upload test validation.");
        }
        WebDriverManager.chromedriver().setup();
        ChromeOptions options = new ChromeOptions();
        options.addArguments("--remote-allow-origins=*");
        options.addArguments("--headless=new");
        options.addArguments("--window-size=1920,1080");
        driver = new ChromeDriver(options);
        driver.manage().window().maximize();
        driver.manage().timeouts().implicitlyWait(Duration.ofSeconds(10));
        wait = new WebDriverWait(driver, Duration.ofSeconds(5));
    }
    @Test
    public void testFileUploadViaSendKeys() {
        // Navigate to the local practice file upload HTML
        String fileUrl = "file:///" + tempHtmlFile.getAbsolutePath().replace("\\", "/");
        System.out.println("Navigating to local HTML: " + fileUrl);
        driver.get(fileUrl);
        // Locating the file upload input element
        System.out.println("Locating upload input field...");
        WebElement uploadInput = wait.until(
            ExpectedConditions.presenceOfElementLocated(By.xpath("//input[@data-testid='file-upload-input']"))
        );
        // Upload using sendKeys strategy with absolute file path
        String uploadFilePath = uploadTxtFile.getAbsolutePath();
        System.out.println("Uploading file: " + uploadFilePath);
        uploadInput.sendKeys(uploadFilePath);
        // Verify status changes to include the uploaded file name
        WebElement statusElement = driver.findElement(By.xpath("//div[@data-testid='upload-status']"));
        String statusText = statusElement.getText();
        System.out.println("Status Message Captured: " + statusText);
        Assert.assertTrue(statusText.contains("Successfully uploaded: upload_test_file.txt"), 
                "File upload status validation failed!");
        System.out.println("File upload test validation completed successfully!");
    }
    @AfterMethod
    public void tearDown() {
        if (driver != null) {
            System.out.println("Quitting driver session...");
            driver.quit();
        }
        // Clean up temporary files
        if (tempHtmlFile != null && tempHtmlFile.exists()) {
            tempHtmlFile.delete();
        }
        if (uploadTxtFile != null && uploadTxtFile.exists()) {
            uploadTxtFile.delete();
        }
    }
}
```

---

## 💻 Test Execution Logs

Below are the clean console reports from compiling and executing the upload suite:

```bash
[INFO] Running com.mycodeyatra.tests.UploadTest
Navigating to local HTML: file:///D:/MyCodeYatra/AILearning2026/Repository/mcyt-sel-java/target/file_upload_demo.html
Locating upload input field...
Uploading file: D:\MyCodeYatra\AILearning2026\Repository\mcyt-sel-java\target\upload_test_file.txt
Status Message Captured: Successfully uploaded: upload_test_file.txt
File upload test validation completed successfully!
Quitting driver session...
Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 4.887 sec
```

---

## 📊 Comparison of Upload Strategies

Choosing the right strategy depends on environment constraints (CI execution vs. local execution).

| Feature Matrix | Selenium `sendKeys()` | Java `Robot` Class | AutoIt Integration |
| :--- | :--- | :--- | :--- |
| **Headless Mode** | ✅ Full Compatibility | ❌ Fails (Requires active GUI display) | ❌ Fails (Requires active GUI display) |
| **OS Compatibility** | ✅ Cross-Platform | ✅ Cross-Platform | ❌ Windows Only |
| **Target Element** | Must be `<input type="file">` | Handles any click container | Handles any click container |
| **Setup Complexity** | Zero (Native Selenium) | Medium (Key event arrays) | High (Requires .exe compilation) |
| **Speed** | Instant | Slow (Simulated typing delay) | Medium (Interpreted script wait) |

---

## ⚠️ Common Pitfalls

* **Using Relative Paths**: `sendKeys("src/test/file.txt")` will throw an `InvalidArgumentException`. Selenium requires the **absolute file path** (e.g. `C:\project\src\test\file.txt`) to locate the source file on the system.
* **Attempting to click the input first**: Running `uploadInput.click()` before invoking `.sendKeys()` opens the native OS window, which freezes test execution headlessly. Invoke `sendKeys` directly on the locator.
* **Hidden `<input>` elements**: Modern web designs hide the input tag (`display: none` or `visibility: hidden`) and render custom div templates. If the input is fully hidden, standard `sendKeys` throws an `ElementNotInteractableException`. To fix this, inject a JavaScript execution command to make the element visible before uploading:
  `((JavascriptExecutor) driver).executeScript("arguments[0].style.display='block';", uploadInput);`

---

## 🙋 Frequently Asked Questions

> **Q: Why does the Robot Class fail in Jenkins/GitLab CI runners?**
>
> **A:** The Java Robot class simulates physical keyboard keys. When running on remote server agents (which run inside virtual display containers), there is no hardware keyboard or active GUI desktop to receive key presses, causing the test to hang or crash.

> **Q: Can we upload multiple files at once using Selenium?**
>
> **A:** Yes, if the input tag contains the `multiple` attribute. You can upload multiple files by joining their absolute paths with a newline character (`\n`) and sending the combined string:
> `uploadInput.sendKeys("path1.txt \n path2.txt");`
