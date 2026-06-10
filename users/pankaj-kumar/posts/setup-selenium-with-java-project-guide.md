---
title: Setup Selenium with Java Project: The Complete Modern Guide
date: 03-Jul-2024
lastUpdated: 10-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [selenium, java, maven, setup]
category: UI Automation
categories: [UI Automation, Selenium, Java]
excerpt: >-
  Setting up an enterprise-grade automation framework requires a clean, repeatable project structure. We set up a project using Maven and the native Selenium Manager.
readTime: 5 min read
---

# Setup Selenium with Java Project: The Complete Modern Guide

> 📅 **Last Updated:** 10-Jun-2026

Setting up an enterprise-grade automation framework requires a clean, repeatable project structure. While setup used to involve manually downloading browser executables (`chromedriver.exe`) or configuring third-party libraries, modern Selenium has simplified this process.

In this guide, we will set up a modern Java test automation project from scratch using **Maven** and the native **Selenium Manager**, and run a test script against our live practice site.

---

## 🚀 What is Selenium Manager?

Introduced in Selenium 4.6, **Selenium Manager** is a native binary (compiled from Rust) bundled directly inside the Selenium library. It automatically:

1. Detects the browser version installed on your local machine.
2. Identifies and downloads the matching driver version (e.g., ChromeDriver).
3. Adds the driver to your execution path.

This means you **no longer need** to manually download drivers or add `webdrivermanager` dependencies to your `pom.xml`.

---

## 🛠️ Step 1: Install JDK and Maven

Before creating your project, ensure you have the following installed:

* **Java Development Kit (JDK 11 or 17)**: Download from OpenJDK or Eclipse Temurin. Set your `JAVA_HOME` environment variable.
* **Apache Maven**: Set up Maven and add its binary path to your system's `PATH`.

---

## 📦 Step 2: Maven Project Structure and pom.xml

Create a standard Maven directory structure. Your project layout should look like this:

```text
mcyt-sel-java/
├── pom.xml
└── src/
    └── test/
        └── java/
            └── com/
                └── mycodeyatra/
                    └── tests/
                        └── SetupTest.java
```

Configure your **[pom.xml](file:///D:/MyCodeYatra/AILearning2026/Repository/mcyt-sel-java/pom.xml)** with the Selenium Java and TestNG dependencies:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.mycodeyatra</groupId>
    <artifactId>mcyt-sel-java</artifactId>
    <version>1.0-SNAPSHOT</version>
    <properties>
        <maven.compiler.source>11</maven.compiler.source>
        <maven.compiler.target>11</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>
    <dependencies>
        <!-- Selenium Java (Contains Selenium Manager natively) -->
        <dependency>
            <groupId>org.seleniumhq.selenium</groupId>
            <artifactId>selenium-java</artifactId>
            <version>4.21.0</version>
        </dependency>
        <!-- TestNG Framework -->
        <dependency>
            <groupId>org.testng</groupId>
            <artifactId>testng</artifactId>
            <version>7.10.2</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
</project>
```

---

## ⚙️ How Selenium Manager Works Under the Hood

The diagram below shows how the Selenium client library communicates with Selenium Manager during project initialization:

![Mermaid Diagram](https://mermaid.ink/img/c2VxdWVuY2VEaWFncmFtCiAgICBwYXJ0aWNpcGFudCBDb2RlIGFzIEphdmEgVGVzdCBDb2RlCiAgICBwYXJ0aWNpcGFudCBMaWJyYXJ5IGFzIFNlbGVuaXVtIEphdmEgTGlicmFyeQogICAgcGFydGljaXBhbnQgTWFuYWdlciBhcyBTZWxlbml1bSBNYW5hZ2VyIChSdXN0IEJpbmFyeSkKICAgIHBhcnRpY2lwYW50IENsb3VkIGFzIENocm9tZS9GaXJlZm94IERyaXZlciBSZXBvCiAgICBwYXJ0aWNpcGFudCBCcm93c2VyIGFzIENocm9tZSBCcm93c2VyCiAgICAKICAgIENvZGUtPj5MaWJyYXJ5OiBuZXcgQ2hyb21lRHJpdmVyKCkKICAgIExpYnJhcnktPj5NYW5hZ2VyOiBRdWVyeSBkcml2ZXIgcGF0aCBmb3IgQ2hyb21lCiAgICBNYW5hZ2VyLT4-QnJvd3NlcjogQ2hlY2sgaW5zdGFsbGVkIGxvY2FsIGJyb3dzZXIgdmVyc2lvbgogICAgQnJvd3Nlci0tPj5NYW5hZ2VyOiBSZXR1cm5zIHZlcnNpb24gKGUuZy4sIHYxMjYpCiAgICBNYW5hZ2VyLT4-Q2xvdWQ6IERvd25sb2FkIG1hdGNoaW5nIENocm9tZURyaXZlciBiaW5hcnkgKHYxMjYpCiAgICBDbG91ZC0tPj5NYW5hZ2VyOiBSZXR1cm5zIGJpbmFyeSBleGVjdXRhYmxlCiAgICBNYW5hZ2VyLS0-PkxpYnJhcnk6IFJldHVybnMgbG9jYWwgcGF0aCB0byBkcml2ZXIgYmluYXJ5CiAgICBMaWJyYXJ5LT4-Q29kZTogTGF1bmNoZXMgQ2hyb21lIGJyb3dzZXIgc3VjY2Vzc2Z1bGx5)

---

## 🚀 Step 3: Writing the Verification Test Script

We will write a test class named `SetupTest.java` that:
1. Launches Chrome using Selenium Manager.
2. Navigates to **`https://practice.mycodeyatra.com/`**.
3. Navigates to the Sandbox Arena and opens the **Form Practice** card.
4. Validates the header text and quits cleanly.

Create **`SetupTest.java`** in your `src/test/java/com/mycodeyatra/tests/` folder:

```java
package com.mycodeyatra.tests;
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
import java.time.Duration;
public class SetupTest {
    private WebDriver driver;
    private WebDriverWait wait;
    @BeforeMethod
    public void setUp() {
        // Configure ChromeOptions to run headlessly
        ChromeOptions options = new ChromeOptions();
        options.addArguments("--remote-allow-origins=*");
        options.addArguments("--headless=new");
        // Initialize ChromeDriver (Selenium Manager discovers the binary automatically)
        driver = new ChromeDriver(options);
        driver.manage().window().maximize();
        driver.manage().timeouts().implicitlyWait(Duration.ofSeconds(10));
        wait = new WebDriverWait(driver, Duration.ofSeconds(5));
    }
    @Test
    public void testProjectSetup() {
        // 1. Navigate to the Live Practice Site
        System.out.println("Navigating to: https://practice.mycodeyatra.com/");
        driver.get("https://practice.mycodeyatra.com/");
        // 2. Open Sandbox Arena Page
        System.out.println("Navigating to Sandbox Arena...");
        WebElement sandboxLink = wait.until(ExpectedConditions.elementToBeClickable(By.xpath("//span[contains(text(),'Sandbox Arena')]")));
        sandboxLink.click();
        // 3. Click the Form Practice Tile Card
        System.out.println("Clicking Form Practice Tile...");
        WebElement formTile = wait.until(ExpectedConditions.elementToBeClickable(By.xpath("//h3[text()='Form Practice']")));
        formTile.click();
        // 4. Validate the Form Header Text
        WebElement formHeader = wait.until(ExpectedConditions.visibilityOfElementLocated(By.xpath("//h2[contains(text(),'Form Submission')]")));
        String headerText = formHeader.getText();
        System.out.println("Loaded Page Header: " + headerText);
        Assert.assertEquals(headerText, "Form Submission & Data Validation", "Failed to navigate to Form Practice page.");
    }
    @AfterMethod
    public void tearDown() {
        // Clean up the driver instance
        if (driver != null) {
            System.out.println("Quitting driver session...");
            driver.quit();
        }
    }
}
```

---

### 📝 Script Code Explanation

| Element / Command | Purpose & Description |
| :--- | :--- |
| **`ChromeOptions`** | An configuration object class used to customize browser settings (such as setting headless mode, disabling extensions, and overriding security properties). |
| **`driver.findElement(By.xpath(...))`** | Locates page elements using locator query strings like XPath. We target specific matching button or header text dynamically on the practice dashboard. |
| **`wait.until(ExpectedConditions...)`** | Explicit wait synchronization rules that pause program execution until the targeted element meets structural rules (such as being clickable or visible). |
| **`driver.quit()`** | Closes all browser windows and kills the underlying driver daemon process to prevent orphan sessions consuming system RAM. |

---

## 📊 Test Execution Output Log

Here is the build success log from executing `mvn test` in our workspace terminal. The test launches, discovers ChromeDriver automatically via Selenium Manager, runs the flow, and completes successfully:

```text
[INFO] Scanning for projects...
[INFO] -------------------< com.mycodeyatra:mcyt-sel-java >--------------------
[INFO] Building mcyt-sel-java 1.0-SNAPSHOT
[INFO] --------------------------------[ jar ]---------------------------------
[INFO] 
[INFO] --- maven-compiler-plugin:3.1:testCompile (default-testCompile) @ mcyt-sel-java ---
[INFO] Compiling 1 source file to D:\MyCodeYatra\AILearning2026\Repository\mcyt-sel-java\target\test-classes
[INFO] 
[INFO] --- maven-surefire-plugin:2.12.4:test (default-test) @ mcyt-sel-java ---
[INFO] Surefire report directory: D:\MyCodeYatra\AILearning2026\Repository\mcyt-sel-java\target\surefire-reports
-------------------------------------------------------
 T E S T S
-------------------------------------------------------
Running com.mycodeyatra.tests.SetupTest
Configuring TestNG with: org.apache.maven.surefire.testng.conf.TestNG652Configurator@35f983a6
Navigating to: https://practice.mycodeyatra.com/
Navigating to Sandbox Arena...
Clicking Form Practice Tile...
Loaded Page Header: Form Submission & Data Validation
Quitting driver session...
Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 6.812 sec
Results :
Tests run: 1, Failures: 0, Errors: 0, Skipped: 0
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  15.112 s
[INFO] Finished at: 2024-07-03T11:00:00+05:30
[INFO] ------------------------------------------------------------------------
```

---

## 💡 Best Practices for Project Configurations

* **Specify Compiler Properties**: Always specify source and target compilation versions (such as `11` or `17`) in your `pom.xml` properties block to avoid standard Maven target version mismatch warnings.
* **Keep Selenium Library Updated**: Since Selenium Manager updates its local driver resolution logic frequently to support browser version rollouts, keeping your parent `selenium-java` package dependency updated is highly recommended.
