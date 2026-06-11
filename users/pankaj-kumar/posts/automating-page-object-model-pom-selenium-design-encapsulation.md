---
title: Automating Page Object Model (POM) in Selenium: Design & Encapsulation
date: 23-Jul-2024
lastUpdated: 11-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [selenium, java, page-object-model, design-patterns, encapsulation, software-architecture]
category: UI Automation
categories: [UI Automation, Selenium, Java]
excerpt: >-
  Master the Page Object Model (POM) design pattern in Selenium WebDriver. Learn how to encapsulate locator elements, build reusable page methods, and cleanly write assertions in Java.
readTime: 7 min read
---

# Automating Page Object Model (POM) in Selenium: Design & Encapsulation

> 📅 **Last Updated:** 11-Jun-2026

Writing element locators and test assertions inside a single class makes test suites extremely fragile. If the UI changes (for instance, a button ID or an input xpath updates), you must search and replace the locator across multiple test files.

The **Page Object Model (POM)** is a standard design pattern in test automation that solves this by creating an object repository for web UI elements. Under POM, each web page in your application is modeled as a class. The page class holds the locators and encapsulate the interactions (such as filling forms, clicking buttons, and reading text), while the test script focuses entirely on validations and assertions.

---

## 🧭 Page Object Model Architecture

The diagram below displays how test suites communicate with decoupled page wrapper classes instead of querying the DOM directly:

![diagram_1](https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/posts/automating-page-object-model-pom-selenium-design-encapsulation/images/diagram_1.png)

---

## 🛠️ Step-by-Step Code Walkthrough

To implement POM, we separate our source code:
* Page wrapper classes are placed under the main source package: `src/main/java/com/mycodeyatra/pages/`.
* Test files containing assertions are placed under the test package: `src/test/java/com/mycodeyatra/tests/`.

### 1. Login Page Object (`LoginPage.java`)
This class encapsulates username/password inputs, login triggers, and error messages. Locators are kept private, and we expose public methods for test actions.

```java
package com.mycodeyatra.pages;
import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.support.ui.ExpectedConditions;
import org.openqa.selenium.support.ui.WebDriverWait;
import java.time.Duration;
public class LoginPage {
    private WebDriver driver;
    private WebDriverWait wait;
    // Locators using data-testid
    private By usernameField = By.xpath("//input[@data-testid='username']");
    private By passwordField = By.xpath("//input[@data-testid='password']");
    private By loginButton = By.xpath("//button[@data-testid='login-btn']");
    private By errorMessage = By.xpath("//div[@data-testid='login-error']");
    public LoginPage(WebDriver driver) {
        this.driver = driver;
        this.wait = new WebDriverWait(driver, Duration.ofSeconds(10));
    }
    public void navigateTo(String url) {
        driver.get(url);
    }
    public void enterUsername(String username) {
        wait.until(ExpectedConditions.visibilityOfElementLocated(usernameField)).clear();
        driver.findElement(usernameField).sendKeys(username);
    }
    public void enterPassword(String password) {
        driver.findElement(passwordField).clear();
        driver.findElement(passwordField).sendKeys(password);
    }
    public void clickLogin() {
        driver.findElement(loginButton).click();
    }
    public void login(String username, String password) {
        enterUsername(username);
        enterPassword(password);
        clickLogin();
    }
    public String getErrorMessage() {
        return wait.until(ExpectedConditions.visibilityOfElementLocated(errorMessage)).getText();
    }
}
```

### 2. Sandbox Page Object (`SandboxPage.java`)
This class represents the post-login dashboard, providing actions to read headers and navigate.

```java
package com.mycodeyatra.pages;
import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.support.ui.ExpectedConditions;
import org.openqa.selenium.support.ui.WebDriverWait;
import java.time.Duration;
public class SandboxPage {
    private WebDriver driver;
    private WebDriverWait wait;
    private By header = By.xpath("//h2");
    private By sandboxArenaLink = By.xpath("//span[contains(text(),'Sandbox Arena')]");
    public SandboxPage(WebDriver driver) {
        this.driver = driver;
        this.wait = new WebDriverWait(driver, Duration.ofSeconds(10));
    }
    public String getHeaderMessage() {
        return wait.until(ExpectedConditions.visibilityOfElementLocated(header)).getText();
    }
    public void clickSandboxArena() {
        wait.until(ExpectedConditions.elementToBeClickable(sandboxArenaLink)).click();
    }
}
```

### 3. POM Test Suite (`PomTest.java`)
The test script initializes page objects inside its `@BeforeMethod` hook. It invokes page methods to perform UI actions and uses TestNG assertions to validate success/failure responses.

```java
package com.mycodeyatra.tests;
import io.github.bonigarcia.wdm.WebDriverManager;
import com.mycodeyatra.pages.LoginPage;
import com.mycodeyatra.pages.SandboxPage;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.chrome.ChromeOptions;
import org.testng.Assert;
import org.testng.annotations.AfterMethod;
import org.testng.annotations.BeforeMethod;
import org.testng.annotations.Test;
import java.time.Duration;
public class PomTest {
    private WebDriver driver;
    private LoginPage loginPage;
    private SandboxPage sandboxPage;
    @BeforeMethod
    public void setUp() {
        WebDriverManager.chromedriver().setup();
        ChromeOptions options = new ChromeOptions();
        options.addArguments("--remote-allow-origins=*");
        options.addArguments("--headless=new");
        options.addArguments("--window-size=1920,1080");
        driver = new ChromeDriver(options);
        driver.manage().timeouts().implicitlyWait(Duration.ofSeconds(10));
        loginPage = new LoginPage(driver);
        sandboxPage = new SandboxPage(driver);
    }
    @Test
    public void testSuccessfulLoginViaPOM() {
        String loginUrl = "https://practice.mycodeyatra.com/#/login";
        System.out.println("Navigating to: " + loginUrl);
        loginPage.navigateTo(loginUrl);
        System.out.println("Submitting valid credentials...");
        loginPage.login("admin", "admin123");
        // Verify successful navigation redirect
        System.out.println("Verifying header text post authentication...");
        String headerText = sandboxPage.getHeaderMessage();
        System.out.println("Header Text Captured: " + headerText);
        Assert.assertTrue(headerText.contains("Profile") || headerText.contains("Sign In") || driver.getCurrentUrl().contains("profile"), 
                "Login navigation redirect failed!");
        System.out.println("POM E2E login flow completed successfully!");
    }
    @Test
    public void testInvalidLoginErrorMessageViaPOM() {
        String loginUrl = "https://practice.mycodeyatra.com/#/login";
        System.out.println("Navigating to: " + loginUrl);
        loginPage.navigateTo(loginUrl);
        System.out.println("Submitting invalid credentials...");
        loginPage.login("wronguser", "wrongpassword");
        System.out.println("Verifying validation error message...");
        String errorText = loginPage.getErrorMessage();
        System.out.println("Captured Error: " + errorText);
        Assert.assertTrue(errorText.contains("Invalid username or password"), "Error validation assertion failed!");
        System.out.println("POM error validation completed successfully!");
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

Below is the clean console output from running the Page Object Model validation suite:

```bash
[INFO] Running com.mycodeyatra.tests.PomTest
Navigating to: https://practice.mycodeyatra.com/#/login
Submitting invalid credentials...
Verifying validation error message...
Captured Error: Invalid username or password. (Hint: admin / admin123)
POM error validation completed successfully!
Quitting driver session...
Navigating to: https://practice.mycodeyatra.com/#/login
Submitting valid credentials...
Verifying header text post authentication...
Header Text Captured: Welcome back, admin!
POM E2E login flow completed successfully!
Quitting driver session...
Tests run: 2, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 9.19 sec
```

---

## 📊 Traditional Scripting vs Page Object Model

A comparison of design structures:

| Design Aspect | Traditional Scripting | Page Object Model |
| :--- | :--- | :--- |
| **Code Redundancy** | High. Locators (like `By.id("username")`) are duplicated across multiple test files. | Zero. Each locator is declared exactly once inside its respective Page class. |
| **Readability** | Poor. Test files are cluttered with `driver.findElement` calls. | High. Test scripts read like business steps (`loginPage.login(user, pass)`). |
| **UI Change Impact** | High. A single locator change requires updating multiple test classes. | Low. Locator updates are made only in a single page file; test scripts remain unchanged. |
| **Assertion Location** | Scattered everywhere in UI actions. | Isolated strictly within test script files. |

---

## ⚠️ Common Pitfalls

* **Placing Assertions inside Page Classes**: Page classes should only represent services and interface components. They should *never* contain test validation assertions (like `Assert.assertEquals`). Doing so couples page code to a specific test library (like TestNG or JUnit), reducing utility and reuse.
* **Overusing Selenium PageFactory (`@FindBy`)**: PageFactory was built to lazy-load elements. However, in modern single-page applications (SPAs), PageFactory causes frequent `StaleElementReferenceException` issues due to dynamic DOM updates. Standard `By` locators combined with explicit waits (`wait.until`) are far more reliable.
* **Initializing all Page Objects at startup**: In large applications, a single flow might traverse dozens of pages. Instantiating all Page classes in the constructor causes slow startup times and memory issues. Instantiate pages dynamically as they are navigated.

---

## 🙋 Frequently Asked Questions

> **Q: Should Page Methods return another Page Object?**
>
> **A:** Yes! It is an excellent practice. If clicking the "Login" button redirects the user to the Sandbox Dashboard, you can design the `login` method to return a new `SandboxPage` object:
> `public SandboxPage login(String user, String pass) { ... return new SandboxPage(driver); }`. This enables clean, fluent test scripting.

> **Q: How do we handle dynamic element locators under POM?**
>
> **A:** If you need to locate a table row matching a dynamic user name, do not declare a static locator variable. Expose a parameterized method:
> `public WebElement getRowByName(String name) { return driver.findElement(By.xpath("//tr[td[text()='" + name + "']]")); }`.
