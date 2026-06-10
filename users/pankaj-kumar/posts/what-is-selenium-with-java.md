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
import java.time.Duration;
public class FirstSeleniumTest {
    public static void main(String[] args) {
        // Setup ChromeDriver binary automatically
        WebDriverManager.chromedriver().setup();
        // Configure headless / browser options if necessary
        ChromeOptions options = new ChromeOptions();
        options.addArguments("--remote-allow-origins=*");
        // Initialize WebDriver instance
        WebDriver driver = new ChromeDriver(options);
        try {
            // 1. Maximize window & configure implicit waits
            driver.manage().window().maximize();
            driver.manage().timeouts().implicitlyWait(Duration.ofSeconds(10));
            // 2. Navigate to the Live Practice Site
            System.out.println("Navigating to target site...");
            driver.get("https://practice.mycodeyatra.com/");
            // 3. Verify Homepage Title
            String homeTitle = driver.getTitle();
            System.out.println("Homepage Title: " + homeTitle);
            // 4. Click Sandbox Arena button to navigate
            System.out.println("Opening Sandbox Arena...");
            WebElement sandboxLink = driver.findElement(By.xpath("//span[contains(text(),'Sandbox Arena')]"));
            sandboxLink.click();
            // 5. Verify navigation to Sandbox by checking the header text
            WebElement header = driver.findElement(By.tagName("h1"));
            String headerText = header.getText();
            System.out.println("Header Text on Sandbox: " + headerText);
            if (headerText.contains("Sandbox Arena")) {
                System.out.println("Test PASSED: Successfully navigated to Sandbox!");
            } else {
                System.out.println("Test FAILED: Header did not match.");
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            // 6. Close the browser session
            System.out.println("Quitting browser session...");
            driver.quit();
        }
    }
}
```

### Script Code Explanation:
1. **`WebDriverManager`**: Eliminates the manual hassle of downloading `chromedriver.exe` and matches the driver version automatically with the installed Chrome version.
2. **`ChromeDriver`**: The class instantiation launches a new Chrome window session.
3. **`driver.get()`**: Commands the browser to load the designated web URL.
4. **`driver.findElement()`**: Finds target WebElements on the page using locator strategies (like XPath and Tag Name).
5. **`driver.quit()`**: Closes all browser windows opened by WebDriver and terminates the driver session. Always invoke this in a `finally` block to prevent orphaned driver processes.

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
