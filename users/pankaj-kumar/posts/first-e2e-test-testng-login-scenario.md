---
title: Writing Your First E2E Test: A Step-by-Step Guide with Selenium and TestNG
date: 11-Jun-2026
lastUpdated: 11-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [selenium, java, testng, e2e, login]
category: UI Automation
categories: [UI Automation, Selenium, Java]
excerpt: >-
  Learn how to write and execute your first End-to-End (E2E) automated login test using Selenium WebDriver, Java, and TestNG, complete with invalid and valid test scenario validations.
readTime: 7 min read
---

# Writing Your First E2E Test: A Step-by-Step Guide with Selenium and TestNG

> 📅 **Last Updated:** 11-Jun-2026

End-to-End (E2E) testing is the ultimate validation layer for any web application. Unlike unit tests that check isolated components or API tests that check headless payloads, E2E tests mimic actual user journeys through the browser interface. 

In this tutorial, we will write and run a complete E2E test for a standard user authentication scenario (Login Flow) using Java, Selenium WebDriver, and the TestNG framework. 

---

## 🧭 The E2E Login Scenario Workflow

An E2E login flow is more than just typing a username and password. It involves handling security layers, validating form validation messages, navigating to profile hubs, and safely terminating sessions.

Here is the exact user journey we will automate:

![diagram_1](https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/posts/first-e2e-test-testng-login-scenario/images/diagram_1.png)

---

## 🛠️ Step-by-Step Code Walkthrough

To build this E2E test, we utilize three critical design paradigms:
1. **Dynamic Locator Strategies**: Using `data-testid` attributes to isolate our locators from CSS styling changes.
2. **Explicit Synchronizations**: Employing `WebDriverWait` to ensure elements are interactable before selenium attempts input.
3. **Multi-phase Assertion Hooks**: Executing negative verification first, followed by positive credential validations to ensure total coverage.

Here is the complete implementation of the E2E login suite:

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
public class LoginTest {
    private WebDriver driver;
    private WebDriverWait wait;
    @BeforeMethod
    public void setUp() {
        // Automatically setup driver binary
        WebDriverManager.chromedriver().setup();
        ChromeOptions options = new ChromeOptions();
        options.addArguments("--remote-allow-origins=*");
        options.addArguments("--headless=new"); // Run headlessly in CI/CD pipeline
        driver = new ChromeDriver(options);
        driver.manage().window().maximize();
        driver.manage().timeouts().implicitlyWait(Duration.ofSeconds(10));
        wait = new WebDriverWait(driver, Duration.ofSeconds(5));
    }
    @Test
    public void testLoginScenario() {
        // 1. Navigate to the Live Practice Site
        System.out.println("Navigating to: https://practice.mycodeyatra.com/");
        driver.get("https://practice.mycodeyatra.com/");
        // 2. Open Sandbox Arena Page
        System.out.println("Navigating to Sandbox Arena...");
        WebElement sandboxLink = wait.until(ExpectedConditions.elementToBeClickable(By.xpath("//span[contains(text(),'Sandbox Arena')]")));
        sandboxLink.click();
        // 3. Click the Authentication & Session Card Tile
        System.out.println("Clicking Authentication & Session Tile...");
        WebElement loginTile = wait.until(ExpectedConditions.elementToBeClickable(By.xpath("//h3[text()='Authentication & Session']")));
        loginTile.click();
        // 4. Validate Sign In Header
        WebElement signInHeader = wait.until(ExpectedConditions.visibilityOfElementLocated(By.xpath("//h2[text()='Sign In']")));
        Assert.assertNotNull(signInHeader, "Sign In page failed to load.");
        // 5. Test Invalid Login
        System.out.println("Testing Invalid Login Credentials...");
        WebElement usernameInput = driver.findElement(By.xpath("//input[@data-testid='username']"));
        WebElement passwordInput = driver.findElement(By.xpath("//input[@data-testid='password']"));
        WebElement loginButton = driver.findElement(By.xpath("//button[@data-testid='login-btn']"));
        usernameInput.clear();
        usernameInput.sendKeys("invalid_user");
        passwordInput.clear();
        passwordInput.sendKeys("wrong_pass");
        loginButton.click();
        // Verify invalid error alert message
        WebElement errorAlert = wait.until(ExpectedConditions.visibilityOfElementLocated(By.xpath("//div[@data-testid='login-error']")));
        String errorText = errorAlert.getText();
        System.out.println("Error Alert Received: " + errorText);
        Assert.assertTrue(errorText.contains("Invalid username or password"), "Incorrect error validation message.");
        // 6. Test Valid Login
        System.out.println("Testing Valid Login Credentials...");
        // Re-locate input fields due to potential DOM refresh
        usernameInput = driver.findElement(By.xpath("//input[@data-testid='username']"));
        passwordInput = driver.findElement(By.xpath("//input[@data-testid='password']"));
        loginButton = driver.findElement(By.xpath("//button[@data-testid='login-btn']"));
        usernameInput.clear();
        usernameInput.sendKeys("admin");
        passwordInput.clear();
        passwordInput.sendKeys("admin123");
        loginButton.click();
        // Verify profile title welcome message
        WebElement profileWelcome = wait.until(ExpectedConditions.visibilityOfElementLocated(By.xpath("//h2[@data-testid='profile-title']")));
        String welcomeMsg = profileWelcome.getText();
        System.out.println("Profile Page Header: " + welcomeMsg);
        Assert.assertTrue(welcomeMsg.contains("Welcome back, admin!"), "Failed to login or load profile page.");
        // Clean logout
        WebElement logoutButton = driver.findElement(By.xpath("//button[@data-testid='logout-btn']"));
        logoutButton.click();
        // Wait back to Sign In header
        signInHeader = wait.until(ExpectedConditions.visibilityOfElementLocated(By.xpath("//h2[text()='Sign In']")));
        Assert.assertNotNull(signInHeader, "Failed to log out safely.");
        System.out.println("Logout complete!");
    }
    @AfterMethod
    public void tearDown() {
        if (driver != null) {
            System.out.println("Quitting driver session...");
            driver.quit();
        }
    }
}
```

---

## 💻 Test Execution Logs

Here is the clean console output from running the E2E Maven execution suite locally:

```bash
[INFO] Running com.mycodeyatra.tests.LoginTest
Navigating to: https://practice.mycodeyatra.com/
Navigating to Sandbox Arena...
Clicking Authentication & Session Tile...
Testing Invalid Login Credentials...
Error Alert Received: Invalid username or password. (Hint: admin / admin123)
Testing Valid Login Credentials...
Profile Page Header: Welcome back, admin!
Logout complete!
Quitting driver session...
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 13.99 s -- in com.mycodeyatra.tests.LoginTest
[INFO] 
[INFO] Results:
[INFO] 
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
```

---

## 📊 Mapping E2E Test Phases

Understanding what happens during each phase of an E2E script helps in organizing your automation framework.

| Test Phase | Purpose | Target Element / Action | Assert Validation |
| :--- | :--- | :--- | :--- |
| **Setup** | Browser configuration & instantiation | ChromeOptions headless setup | Driver is active |
| **Navigation** | Navigate to login form | Click sandbox, click Auth Card | `Sign In` header is visible |
| **Negative Validation** | Test incorrect credentials | Submit wrong credentials | `login-error` shows invalid text |
| **Positive Validation** | Test valid auth pipeline | Submit `admin`/`admin123` | `profile-title` displays "Welcome back, admin!" |
| **Teardown** | Session cleanup | Click logout button, invoke `.quit()` | Redirect to login page, closed socket |

---

## ⚠️ Common Pitfalls to Avoid

* **Hardcoded Credentials**: Avoid hardcoding `admin` and `admin123` directly in your test suites. Read them from configuration files or environment variables instead.
* **Failing to Re-locate Elements**: Dynamic frameworks (like React or Angular) replace DOM elements after state shifts. Re-locate the username input fields before the positive test phase to prevent `StaleElementReferenceException`.
* **Missing Logout Actions**: Failing to log out leaves session tokens active in local storage. Always conclude your tests with a clean logout state.

---

## 🙋 Frequently Asked Questions

> **Q: Why should we use explicit waits over implicit waits in login E2E scenarios?**
>
> **A:** Login forms often involve transitions, redirects, and animations. Implicit waits only check for element presence in the DOM, whereas explicit waits like `elementToBeClickable` ensure the element has completed transition animations and is ready for text input.

> **Q: What is a `data-testid` attribute?**
>
> **A:** A dedicated custom HTML attribute (`data-testid="username"`) added specifically for automated testing. Selecting elements via test-ids makes scripts robust because developers can change CSS classes or page structures without breaking the underlying tests.
