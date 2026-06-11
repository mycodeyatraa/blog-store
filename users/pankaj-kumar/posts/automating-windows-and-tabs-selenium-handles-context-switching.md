---
title: Automating Windows and Tabs in Selenium WebDriver: Handles & Context Switching
date: 16-Jul-2024
lastUpdated: 11-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [selenium, java, window-handles, multiple-tabs, context-switching, selenium-4]
category: UI Automation
categories: [UI Automation, Selenium, Java]
excerpt: >-
  A comprehensive tutorial on managing multiple windows and browser tabs in Selenium using Java. Master window handles, switching techniques, and Selenium 4's newWindow API.
readTime: 7 min read
---

# Automating Windows and Tabs in Selenium WebDriver: Handles & Context Switching

> 📅 **Last Updated:** 11-Jun-2026

Modern web applications frequently open links in new tabs or trigger pop-up windows. In Selenium, every browser window or tab is assigned a unique alphanumeric identifier called a **Window Handle**.

Unlike a user who can visually click and interact with any tab, Selenium is tied to a single execution context at a time. To interact with elements on a new tab or window, you must explicitly fetch the target handle and instruct the driver to switch its context.

---

## 🔄 Window Context Switching Pipeline

The lifecycle of window handles involves tracking the parent window handle, triggering the action that opens a new tab, querying all open window handles, switching context to the child window, and returning back to the parent once completed.

![diagram_1](https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/posts/automating-windows-and-tabs-selenium-handles-context-switching/images/diagram_1.png)

---

## 🛠️ Step-by-Step Code Walkthrough

To automate window handling, we use the following native methods:
1. `driver.getWindowHandle()`: Retrieves the unique handle ID of the current active window.
2. `driver.getWindowHandles()`: Retrieves a `Set<String>` containing all active window handles under the current driver session.
3. `driver.switchTo().window(String nameOrHandle)`: Switches the driver's execution focus to the specified window or tab.
4. `driver.switchTo().newWindow(WindowType type)`: A Selenium 4 feature that opens a new blank tab or window and automatically switches the context.

Here is the complete TestNG implementation demonstrating these techniques:

```java
package com.mycodeyatra.tests;
import io.github.bonigarcia.wdm.WebDriverManager;
import org.openqa.selenium.By;
import org.openqa.selenium.PageLoadStrategy;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.WindowType;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.chrome.ChromeOptions;
import org.openqa.selenium.support.ui.ExpectedConditions;
import org.openqa.selenium.support.ui.WebDriverWait;
import org.testng.Assert;
import org.testng.annotations.AfterMethod;
import org.testng.annotations.BeforeMethod;
import org.testng.annotations.Test;
import java.time.Duration;
import java.util.Set;
public class WindowTabTest {
    private WebDriver driver;
    private WebDriverWait wait;
    @BeforeMethod
    public void setUp() {
        WebDriverManager.chromedriver().setup();
        ChromeOptions options = new ChromeOptions();
        options.addArguments("--remote-allow-origins=*");
        options.addArguments("--headless=new");
        options.addArguments("--window-size=1920,1080");
        options.setPageLoadStrategy(PageLoadStrategy.NONE);
        driver = new ChromeDriver(options);
        driver.manage().timeouts().implicitlyWait(Duration.ofSeconds(10));
        wait = new WebDriverWait(driver, Duration.ofSeconds(10));
    }
    @Test
    public void testSwitchBetweenWindows() {
        String practiceUrl = "https://practice.mycodeyatra.com/#/frames";
        System.out.println("Navigating to Live Practice Site: " + practiceUrl);
        driver.get(practiceUrl);
        // Get the parent window handle
        String parentWindowHandle = driver.getWindowHandle();
        System.out.println("Parent Window Handle: " + parentWindowHandle);
        // Locate and click the button to open a new tab
        WebElement openTabButton = wait.until(
            ExpectedConditions.elementToBeClickable(By.xpath("//button[@data-testid='open-tab-btn']"))
        );
        System.out.println("Clicking button to open child window/tab...");
        openTabButton.click();
        // Wait for the new window to be opened (total 2 windows)
        wait.until(ExpectedConditions.numberOfWindowsToBe(2));
        // Get all window handles
        Set<String> allWindowHandles = driver.getWindowHandles();
        System.out.println("Total open windows: " + allWindowHandles.size());
        // Switch to the child window
        boolean switched = false;
        for (String handle : allWindowHandles) {
            if (!handle.equals(parentWindowHandle)) {
                System.out.println("Switching to Child Window Handle: " + handle);
                driver.switchTo().window(handle);
                switched = true;
                break;
            }
        }
        Assert.assertTrue(switched, "Failed to switch to the child tab!");
        // Assert child window title and page content
        String childTitle = driver.getTitle();
        System.out.println("Child Title: " + childTitle);
        Assert.assertEquals(childTitle, "Practice Tab");
        WebElement childHeader = wait.until(ExpectedConditions.presenceOfElementLocated(By.xpath("//h1")));
        Assert.assertEquals(childHeader.getText(), "Success!");
        // Close the child window
        System.out.println("Closing child window/tab...");
        driver.close();
        // Switch back to parent window
        System.out.println("Switching back to Parent Window...");
        driver.switchTo().window(parentWindowHandle);
        // Assert parent page is active again
        String parentTitle = driver.getTitle();
        System.out.println("Parent Title: " + parentTitle);
        Assert.assertTrue(parentTitle.contains("MyCodeYatra"), "Failed to return to parent page");
    }
    @Test
    public void testSelenium4NewWindowFeature() {
        String practiceUrl = "https://practice.mycodeyatra.com/#/frames";
        System.out.println("Navigating to Live Practice Site: " + practiceUrl);
        driver.get(practiceUrl);
        String parentHandle = driver.getWindowHandle();
        // Selenium 4: Open a new blank tab and switch automatically
        System.out.println("Opening a new blank tab and switching context automatically...");
        driver.switchTo().newWindow(WindowType.TAB);
        driver.get("https://practice.mycodeyatra.com/#/login");
        Assert.assertTrue(driver.getTitle().contains("MyCodeYatra"));
        WebElement loginHeader = wait.until(ExpectedConditions.presenceOfElementLocated(By.xpath("//h2")));
        Assert.assertTrue(loginHeader.getText().contains("Sign In"), "Failed to load Sign In page in new tab");
        // Close new tab and go back
        driver.close();
        driver.switchTo().window(parentHandle);
        Assert.assertTrue(driver.getTitle().contains("MyCodeYatra"));
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

Below is the clean console output from compiling and running the window switching validation suite:

```bash
[INFO] Running com.mycodeyatra.tests.WindowTabTest
Navigating to Live Practice Site: https://practice.mycodeyatra.com/#/frames
Opening a new blank tab and switching context automatically...
Quitting driver session...
Navigating to Live Practice Site: https://practice.mycodeyatra.com/#/frames
Parent Window Handle: 21FCEDC6C1F25F50B1EA0F8C33AB8584
Clicking button to open child window/tab...
Total open windows: 2
Switching to Child Window Handle: 566BDFC90F9A9E4A054378A2ADEDA200
Child Title: Practice Tab
Closing child window/tab...
Switching back to Parent Window...
Parent Title: MyCodeYatra | Test Automation Sandbox
Quitting driver session...
Tests run: 2, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 8.167 sec
```

---

## 📊 Traditional Switching vs Selenium 4 `newWindow` API

| Action / Capability | Traditional Window Handles | Selenium 4 `newWindow()` API |
| :--- | :--- | :--- |
| **Tab Initiation Trigger** | User action (e.g., clicking on a link with `target='_blank'`) | Explicit API directive (`driver.switchTo().newWindow()`) |
| **Focus Switching** | Manual iteration over `getWindowHandles()` and checking IDs | Automated context switch to the new window/tab immediately |
| **New Window Creation** | Relies on application state changes or executing JS scripts | Native command to launch a clean blank tab or window |
| **Use Case** | Validating user-driven popups, external links, social logins | Navigating to diagnostic dashboard pages, comparing data in side-by-side tabs |

---

## ⚠️ Common Pitfalls

* **Closing the parent window instead of the child**: Always pay attention to which window handle is currently active. Calling `driver.close()` terminates the tab/window under active context. If you close the child window, you must still explicitly invoke `driver.switchTo().window(parentHandle)` before executing any further action, otherwise Selenium will throw a `NoSuchWindowException`.
* **Assuming ordered elements in `getWindowHandles()`**: The Set returned by `driver.getWindowHandles()` does not guarantee any order of elements. Never assume that the last element in the Set is the newly opened window. Always filter using condition checks against your tracked parent handle.
* **Failing to wait for the window count to increase**: When clicking on slow links that spawn tabs, `driver.getWindowHandles()` may be checked before the browser has completed launching the tab. Use `ExpectedConditions.numberOfWindowsToBe(int expectedNumberOfWindows)` to ensure the browser processes the new window context before executing switching logic.
