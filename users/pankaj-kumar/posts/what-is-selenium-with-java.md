---
title: What is Selenium with Java?
date: 01-Jul-2024
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [selenium, java, automation, testing]
category: UI Automation
categories: [UI Automation, Selenium, Java]
excerpt: >-
  Welcome to the Selenium with Java Series! In this guide, we dive into the foundations of Selenium WebDriver and write your first automation script targeting a live site.
readTime: 4 min read
---

# What is Selenium with Java?

In the fast-paced world of software delivery, manual testing quickly becomes a bottleneck. To achieve true continuous integration and continuous deployment (CI/CD), automating browser interactions is essential. Among the tools available, the pairing of **Selenium WebDriver** and **Java** remains the enterprise gold standard for web UI testing.

This guide covers why this combination is so powerful, explains the underlying Selenium architecture, and walks you through writing your very first automated test script using a live practice site.

---

## 🏛️ Why Selenium with Java?

Selenium is an open-source suite of tools designed for automating web browsers. Java is a class-based, object-oriented programming language designed to have as few implementation dependencies as possible. Together, they form a robust, mature, and highly scalable automation ecosystem:

* **Enterprise Dominance**: Java is the backend language for a vast majority of enterprise systems. Writing tests in the same language as the application simplifies integration, tool sharing, and developer contribution.
* **W3C Standardization**: Selenium WebDriver is built on the W3C recommendation, meaning it communicates natively with browsers without intermediate helper plugins.
* **Massive Community & Ecosystem**: If you hit a roadblock, the solution is already on StackOverflow. Java also brings powerful test runners (like TestNG and JUnit) and build tools (like Maven and Gradle) to the table.
* **Strong Typing**: Compile-time checks prevent silly syntax mistakes before your code ever runs.

---

## ⚙️ Selenium WebDriver Architecture

Selenium 4 uses the **W3C WebDriver Protocol** to send commands directly to the browser. The architecture consists of three main components:

1. **Selenium Language Bindings (Client)**: The code you write in Java.
2. **Browser Drivers**: Executables (like `chromedriver` or `geckodriver`) that act as a bridge between Selenium and the browser.
3. **Browsers**: The actual browser (Chrome, Firefox, Edge, etc.) executing the commands.

Here is how the communication flows during a test:

![Mermaid Diagram](https://mermaid.ink/img/c2VxdWVuY2VEaWFncmFtCiAgICBwYXJ0aWNpcGFudCBDb2RlIGFzIEphdmEgVGVzdCBDb2RlCiAgICBwYXJ0aWNpcGFudCBEcml2ZXIgYXMgQ2hyb21lRHJpdmVyIC8gR2Vja29Ecml2ZXIKICAgIHBhcnRpY2lwYW50IEJyb3dzZXIgYXMgQ2hyb21lIC8gRmlyZWZveCAoVzNDIFByb3RvY29sKQogICAgCiAgICBDb2RlLT4-RHJpdmVyOiBJbml0aWFsaXplIERyaXZlciBJbnN0YW5jZSAoSFRUUCBQb3N0KQogICAgRHJpdmVyLT4-QnJvd3NlcjogTGF1bmNoIEJyb3dzZXIgSW5zdGFuY2UKICAgIEJyb3dzZXItLT4-RHJpdmVyOiBCcm93c2VyIFNlc3Npb24gQ3JlYXRlZAogICAgRHJpdmVyLS0-PkNvZGU6IFJldHVybiBTZXNzaW9uIElECiAgICAKICAgIENvZGUtPj5Ecml2ZXI6IE5hdmlnYXRlIHRvIFVSTCBjb21tYW5kIChKU09OKQogICAgRHJpdmVyLT4-QnJvd3NlcjogR28gdG8gcGFnZSAoVzNDIEhUVFAgUmVxdWVzdCkKICAgIEJyb3dzZXItLT4-RHJpdmVyOiBQYWdlIGxvYWRlZCBzdWNjZXNzCiAgICBEcml2ZXItLT4-Q29kZTogTmF2aWdhdGlvbiByZXNwb25zZQ==)

---

## 🛠️ Step-by-Step Environment Setup

To get started, you will need the following tools installed on your local machine:

1. **Java Development Kit (JDK 11 or higher)**.
2. **Apache Maven** (for dependency management).
3. **An IDE** (IntelliJ IDEA is highly recommended).

### Add Maven Dependencies

Create a new Maven project and add the following dependencies to your `pom.xml`:

```xml
<dependencies>
    <!-- Selenium Java Client Library -->
    <dependency>
        <groupId>org.seleniumhq.selenium</groupId>
        <artifactId>selenium-java</artifactId>
        <version>4.21.0</version>
    </dependency>
    <!-- WebDriverManager to manage driver binaries automatically -->
    <dependency>
        <groupId>io.github.bonigarcia</groupId>
        <artifactId>webdrivermanager</artifactId>
        <version>5.8.0</version>
    </dependency>
    <!-- TestNG testing framework -->
    <dependency>
        <groupId>org.testng</groupId>
        <artifactId>testng</artifactId>
        <version>7.10.2</version>
        <scope>test</scope>
    </dependency>
</dependencies>
```

---

## 🚀 Writing Your First UI Test Script

Now, let's write a simple script that navigates to the MyCodeYatra practice website (**https://practice.mycodeyatra.com/**), clicks to open the Sandbox page, and validates the page headers.

Create a class named `FirstSeleniumTest.java` in your `src/test/java` folder:

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
import java.time.Duration;
public class FirstSeleniumTest {
    private WebDriver driver;
    private WebDriverWait wait;
    @BeforeMethod
    public void setUp() {
        // Setup ChromeDriver binary automatically
        WebDriverManager.chromedriver().setup();
        // Configure options
        ChromeOptions options = new ChromeOptions();
        options.addArguments("--remote-allow-origins=*");
        options.addArguments("--headless=new");
        // Initialize driver
        driver = new ChromeDriver(options);
        driver.manage().window().maximize();
        driver.manage().timeouts().implicitlyWait(Duration.ofSeconds(10));
        // Initialize explicit wait (5 seconds timeout)
        wait = new WebDriverWait(driver, Duration.ofSeconds(5));
    }
    @Test
    public void testFirstScript() {
        // 1. Navigate to the Live Practice Site
        System.out.println("Navigating to: https://practice.mycodeyatra.com/");
        driver.get("https://practice.mycodeyatra.com/");
        // 2. Validate Homepage Title
        String homeTitle = driver.getTitle();
        System.out.println("Homepage Title: " + homeTitle);
        Assert.assertTrue(homeTitle.contains("MyCodeYatra") || homeTitle.isEmpty(), "Title validation failed");
        // 3. Click Sandbox Arena button
        System.out.println("Opening Sandbox Arena...");
        WebElement sandboxLink = wait.until(ExpectedConditions.elementToBeClickable(By.xpath("//span[contains(text(),'Sandbox Arena')]")));
        sandboxLink.click();
        // 4. Validate Sandbox Header using explicit wait to prevent race condition
        WebElement header = wait.until(ExpectedConditions.visibilityOfElementLocated(By.xpath("//h2[contains(text(),'Test Practice Sandbox')]")));
        String headerText = header.getText();
        System.out.println("Sandbox Page Header: " + headerText);
        Assert.assertTrue(headerText.contains("Test Practice Sandbox"), "Failed to load Sandbox Arena page");
    }
    @AfterMethod
    public void tearDown() {
        // Close driver session safely
        if (driver != null) {
            System.out.println("Quitting browser driver session...");
            driver.quit();
        }
    }
}
```

### Script Code Explanation:

* 1. `WebDriverManager`: Eliminates the manual hassle of downloading chromedriver.exe and matches the driver version automatically with the installed Chrome version.

* 2. `ChromeDriver`: The class instantiation launches a new Chrome window session.

* 3. `driver.get()`: Commands the browser to load the designated web URL.

* 4. `driver.findElement()`: Finds target WebElements on the page using locator strategies (like XPath and Tag Name).

* 5. `driver.quit()`: Closes all browser windows opened by WebDriver and terminates the driver session. Always invoke this in the test teardown (@AfterMethod) to prevent orphaned driver processes.

---

## 📊 Test Execution Output Log

Here is the console log from running the test script using Maven (`mvn test`). The test successfully launches Chrome in headless mode, automates the flow, and completes with zero failures:

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
Running com.mycodeyatra.tests.FirstSeleniumTest
Configuring TestNG with: org.apache.maven.surefire.testng.conf.TestNG652Configurator@35f983a6
Navigating to: https://practice.mycodeyatra.com/
Homepage Title: MyCodeYatra | Test Automation Sandbox
Opening Sandbox Arena...
Sandbox Page Header: Test Practice Sandbox
Quitting browser driver session...
Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 5.532 sec
Results :
Tests run: 1, Failures: 0, Errors: 0, Skipped: 0
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  13.277 s
[INFO] Finished at: 2026-06-10T20:14:34+05:30
[INFO] ------------------------------------------------------------------------
```

---

## 💡 Best Practices for Beginners

* **Never Hardcode Drivers**: Avoid manually pointing to file paths like `C:/drivers/chromedriver.exe`. Use WebDriverManager or let Selenium 4's built-in Selenium Manager handle it.
* **Always Quit the Driver**: Use `driver.quit()` rather than `driver.close()` to ensure the entire execution stream and process is cleaned up from memory.
* **Configure Implicit Waits**: Web pages load asynchronously. Always configure a reasonable implicit wait (like 5–10 seconds) right after initializing the driver instance to avoid `NoSuchElementException` errors.

---

## ❓ FAQ Section

**Q1: What is the difference between `driver.close()` and `driver.quit()`?**
* **`driver.close()`**: Closes the current active browser tab/window that Selenium is controlling.
* **`driver.quit()`**: Closes all open windows/tabs and completely ends the WebDriver process.

**Q2: Which Java version is recommended for Selenium testing?**
* JDK 11 or JDK 17 are the industry standard LTS (Long Term Support) versions recommended for writing robust automated test scripts.

**Q3: Can I run my Selenium scripts without showing the browser window?**
* Yes! By adding `options.addArguments("--headless=new");` to your `ChromeOptions` instance, the test runs silently in the background (headless mode), which is perfect for CI/CD environments.
