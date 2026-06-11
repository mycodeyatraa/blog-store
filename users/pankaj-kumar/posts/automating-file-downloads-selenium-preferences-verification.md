---
title: Automating File Downloads in Selenium: Preferences and Verification
date: 15-Jul-2024
lastUpdated: 11-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [selenium, java, file-download, browser-preferences, chrome-options, validation]
category: UI Automation
categories: [UI Automation, Selenium, Java]
excerpt: >-
  A complete guide to automating file downloads in Selenium WebDriver. Learn how to configure browser preferences to download files to custom folders headlessly in Java.
readTime: 7 min read
---

# Automating File Downloads in Selenium: Preferences and Verification

> 📅 **Last Updated:** 11-Jun-2026

Automating file downloads is a critical scenario for testing document exports, PDF generation, or CSV reports. However, by default, web browsers prompt the user with native dialog boxes to select the target download location. 

Because Selenium cannot click native OS pop-ups, your scripts will freeze waiting for user interaction. In this tutorial, we will configure Chrome browser preferences to bypass download prompts, direct files to a custom folder, and write a verification loop in Java.

---

## 🧭 Browser Download Configuration Pipeline

The lifecycle of an automated file download requires configuring preferences before the session starts, intercepting browser download actions, and checking the system directory for completion.

![diagram_1](https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/posts/automating-file-downloads-selenium-preferences-verification/images/diagram_1.png)

---

## 🛠️ Step-by-Step Code Walkthrough

To automate file downloads, we utilize Chrome Experimental Options to:
1. Set `download.default_directory` to map downloads directly into our target project directory.
2. Disable the native save prompts by setting `download.prompt_for_download` to `false`.
3. Set up a polling cycle to wait until the file is completely written on the disk.

Here is the complete TestNG implementation:

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
import java.util.HashMap;
import java.util.Map;
public class DownloadTest {
    private WebDriver driver;
    private WebDriverWait wait;
    private File downloadFolder;
    @BeforeMethod
    public void setUp() throws IOException {
        File targetDir = new File("target");
        if (!targetDir.exists()) {
            targetDir.mkdirs();
        }
        // Configure custom local download folder
        downloadFolder = new File(targetDir, "downloads");
        if (!downloadFolder.exists()) {
            downloadFolder.mkdirs();
        }
        WebDriverManager.chromedriver().setup();
        ChromeOptions options = new ChromeOptions();
        options.addArguments("--remote-allow-origins=*");
        options.addArguments("--headless=new");
        options.addArguments("--window-size=1920,1080");
        // Set Chrome preferences for automatic download directory
        Map<String, Object> prefs = new HashMap<>();
        prefs.put("download.default_directory", downloadFolder.getAbsolutePath());
        prefs.put("download.prompt_for_download", false);
        prefs.put("safebrowsing.enabled", true);
        options.setExperimentalOption("prefs", prefs);
        driver = new ChromeDriver(options);
        driver.manage().window().maximize();
        driver.manage().timeouts().implicitlyWait(Duration.ofSeconds(10));
        wait = new WebDriverWait(driver, Duration.ofSeconds(10));
    }
    @Test
    public void testFileDownloadAndVerify() throws InterruptedException {
        // Navigate to the live practice file download URL
        String fileUrl = "https://practice.mycodeyatra.com/#/upload-download";
        System.out.println("Navigating to live URL: " + fileUrl);
        driver.get(fileUrl);
        // Define expected custom text to write and verify
        String expectedText = "Hello MyCodeYatra Custom Download! Verification text contents.";
        // Locate the custom download text input box
        System.out.println("Locating text input box...");
        WebElement textInput = wait.until(
            ExpectedConditions.visibilityOfElementLocated(By.xpath("//textarea[@data-testid='download-text-input']"))
        );
        textInput.clear();
        textInput.sendKeys(expectedText);
        // Locating the download button
        System.out.println("Locating download button...");
        WebElement downloadBtn = wait.until(
            ExpectedConditions.elementToBeClickable(By.xpath("//button[@data-testid='download-btn']"))
        );
        // Click the download link to trigger file save
        System.out.println("Clicking download button to trigger file save...");
        downloadBtn.click();
        // Poll target download directory until the file exists and download is completed
        File downloadedFile = new File(downloadFolder, "dynamic_download.txt");
        System.out.println("Polling for downloaded file at: " + downloadedFile.getAbsolutePath());
        boolean fileDownloaded = false;
        // Wait maximum 10 seconds (20 iterations of 500ms sleep)
        for (int i = 0; i < 20; i++) {
            if (downloadedFile.exists() && downloadedFile.length() > 0) {
                fileDownloaded = true;
                break;
            }
            Thread.sleep(500);
        }
        Assert.assertTrue(fileDownloaded, "Downloaded file was not found or remained empty!");
        System.out.println("File successfully downloaded! Size: " + downloadedFile.length() + " bytes.");
        // Read file contents and verify matching text
        String content = "";
        try {
            content = new String(java.nio.file.Files.readAllBytes(downloadedFile.toPath()));
        } catch (IOException e) {
            Assert.fail("Failed to read downloaded file: " + e.getMessage());
        }
        System.out.println("Downloaded Content: " + content);
        Assert.assertEquals(content, expectedText, "Downloaded file content does not match input!");
        // Clean validation completed
        System.out.println("File download test validation completed successfully!");
    }
    @AfterMethod
    public void tearDown() {
        if (driver != null) {
            System.out.println("Quitting driver session...");
            driver.quit();
        }
        // Clean up download directory
        if (downloadFolder != null && downloadFolder.exists()) {
            File[] files = downloadFolder.listFiles();
            if (files != null) {
                for (File file : files) {
                    file.delete();
                }
            }
            downloadFolder.delete();
        }
    }
}
```

---

## 💻 Test Execution Logs

Below is the clean console output from compiling and running the download validation suite:

```bash
[INFO] Running com.mycodeyatra.tests.DownloadTest
Navigating to live URL: https://practice.mycodeyatra.com/#/upload-download
Locating text input box...
Locating download button...
Clicking download button to trigger file save...
Polling for downloaded file at: D:\MyCodeYatra\AILearning2026\Repository\mcyt-sel-java\target\downloads\dynamic_download.txt
File successfully downloaded! Size: 62 bytes.
Downloaded Content: Hello MyCodeYatra Custom Download! Verification text contents.
File download test validation completed successfully!
Quitting driver session...
Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 4.887 sec
```

---

## 📊 Chrome Preferences for Downloads

Configuring Chrome options requires injecting specific keys inside the preferences HashMap.

| Preferences Key | Target Value Type | Functionality Description |
| :--- | :--- | :--- |
| `download.default_directory` | String (Absolute Path) | Sets the default folder where downloads are saved |
| `download.prompt_for_download` | Boolean (`false`) | Disables browser native save-as dialog prompts |
| `safebrowsing.enabled` | Boolean (`true`) | Prevents browser warning alerts on suspicious extensions |
| `download.directory_upgrade` | Boolean (`true`) | Forces chrome to upgrade folder mappings without prompts |

---

## ⚠️ Common Pitfalls

* **Checking for files too early (Race Conditions)**: Checking for the file immediately after invoking `click()` will fail. Browser file transfers take time. During the transfer, Chrome creates a temporary file with the suffix `.crdownload` (e.g. `report.pdf.crdownload`). Always write a polling loop to verify the final file exists and is not empty.
* **Failing to clean the download directory**: If your test asserts the presence of a file named `report.csv` in your downloads folder, and a past test run failed to delete it, subsequent runs will pass falsely even if the download failed. Always delete or clean your download directory inside `@BeforeMethod` or `@AfterMethod` teardown hooks.
* **Incorrect Windows Path Formatting**: In Java, file separators on Windows require escaping (e.g., `D:\\MyCodeYatra\\target`). Standardize path formats using `downloadFolder.getAbsolutePath()` to prevent system exceptions.

---

## 🙋 Frequently Asked Questions

> **Q: How can we download files headlessly in Chrome?**
>
> **A:** Chrome options preferences configuration is completely compatible with headless execution. In older ChromeDriver versions, headless execution blocked file transfers, but this is resolved in Chrome's modern headless engine (`--headless=new`).

> **Q: Is it possible to assert the downloaded file contents (like reading PDFs or Excel)?**
>
> **A:** Yes. Once Selenium completes the download, you can use standard Java library utilities (like Apache POI for Excel or PDFBox for PDFs) to query and parse the downloaded file directly from your local filesystem.
