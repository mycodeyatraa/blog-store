---
title: Capturing Failure: Advanced Allure Integrations in Selenium
date: 15-Aug-2026
lastUpdated: 15-Aug-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [java, selenium, allure, testng, listeners, screenshots, reporting, test-automation]
category: Selenium Java
categories: [Selenium Java, Reporting & Analytics]
excerpt: >-
  A red dashboard is useless without context. Learn how to implement TestNG ITestListeners to automatically capture Selenium screenshots and page sources the exact millisecond a test fails, and attach them directly to your Allure report.
readTime: 6 min read
---

# Capturing Failure: Advanced Allure Integrations in Selenium

In our previous tutorial, we learned the basics of the Allure Framework. We configured the `maven-surefire-plugin` and used annotations like `@Epic` and `@Step` to organize our passing tests into a beautiful executive dashboard.

But what happens when a test fails?

A red bar on a dashboard tells a Project Manager that something is broken. But it doesn't help the Automation Engineer fix the bug. To debug effectively, we need **context**. We need to know exactly what the browser looked like the millisecond the test failed.

In this tutorial, we will dive into **Advanced Allure**. We will learn how to implement a TestNG Listener to automatically intercept failures, capture a Selenium screenshot, extract the browser's DOM source code, and attach them directly to the Allure Report!

---

## 1. The Magic of TestNG Listeners

You should *never* write `try-catch` blocks in every single test method to take screenshots. That violates the DRY (Don't Repeat Yourself) principle and clutters your business logic.

Instead, TestNG provides an interface called `ITestListener`. By implementing this interface, we can write a single block of code that "listens" to the entire test suite and executes automatically whenever any test fails.

Create a new class named `TestFailureListener`:

```java
import io.qameta.allure.Attachment;
import org.openqa.selenium.OutputType;
import org.openqa.selenium.TakesScreenshot;
import org.openqa.selenium.WebDriver;
import org.testng.ITestListener;
import org.testng.ITestResult;
public class TestFailureListener implements ITestListener {
    // This method fires automatically the instant an Assert fails!
    @Override
    public void onTestFailure(ITestResult result) {
        System.out.println("*** Test Failed: " + result.getName() + " ***");
        // 1. We must retrieve the WebDriver instance from the failed test class!
        // (Assuming your BaseTest class exposes the driver via a getter)
        Object testClass = result.getInstance();
        WebDriver driver = ((BaseTest) testClass).getDriver();
        if (driver != null) {
            // 2. Capture the Screenshot and attach to Allure
            attachScreenshotToAllure(driver);
            // 3. Capture the DOM Source and attach to Allure
            attachPageSourceToAllure(driver);
        }
    }
    // The @Attachment annotation tells Allure to embed the returned byte array into the report!
    @Attachment(value = "Failure Screenshot", type = "image/png")
    public byte[] attachScreenshotToAllure(WebDriver driver) {
        return ((TakesScreenshot) driver).getScreenshotAs(OutputType.BYTES);
    }
    @Attachment(value = "Page DOM Source", type = "text/html")
    public String attachPageSourceToAllure(WebDriver driver) {
        return driver.getPageSource();
    }
}
```

---

## 2. Registering the Listener

TestNG doesn't know about your custom Listener until you explicitly register it.

There are two ways to do this:

**Option 1: In the testng.xml file (Recommended)**
This applies the listener globally to your entire suite.

```xml
<!DOCTYPE suite SYSTEM "https://testng.org/testng-1.0.dtd" >
<suite name="E2E Automation Suite">
    <!-- Register the custom Listener -->
    <listeners>
        <listener class-name="com.mycodeyatra.listeners.TestFailureListener" />
    </listeners>
    <test name="Regression Tests">
        <classes>
            <class name="com.mycodeyatra.tests.LoginTest" />
        </classes>
    </test>
</suite>
```

**Option 2: Via Class Annotations**
If you only want the listener active for specific classes, you can annotate the test class directly.

```java
@Listeners(com.mycodeyatra.listeners.TestFailureListener.class)
public class LoginTest extends BaseTest {
    // ... test methods
}
```

---

## 3. Writing a Failing Test

Let's write a test that is guaranteed to fail, just so we can see our Listener in action!

```java
import org.openqa.selenium.By;
import org.testng.Assert;
import org.testng.annotations.Test;
public class LoginTest extends BaseTest {
    @Test(description = "Verify login fails with invalid credentials")
    public void testInvalidLoginFails() {
        driver.get("https://practice.mycodeyatra.com/login");
        driver.findElement(By.id("user")).sendKeys("hacker");
        driver.findElement(By.id("pass")).sendKeys("badpassword");
        driver.findElement(By.id("submit")).click();
        // This Assert will FAIL because the ID is actually 'error-msg', not 'wrong-id'!
        Assert.assertTrue(driver.findElement(By.id("wrong-id")).isDisplayed(),
            "Error message was not displayed!");
    }
}
```

---

## 4. Viewing the Attached Evidence

Run your suite via `mvn clean test`, and then execute `allure serve allure-results` in your terminal.

When the Allure dashboard opens, click on the **"Suites"** or **"Behaviors"** tab and find your failed `testInvalidLoginFails` test. 

Expand the test details, and right beneath the massive Java Stacktrace, you will see a new section called **"Attachments"**.

Clicking on the attachments will reveal:
1. A crisp, full-resolution `.png` screenshot showing exactly what the screen looked like when the `NoSuchElementException` was thrown!
2. The raw HTML source code of the page, allowing you to inspect the DOM and verify that the ID really was `error-msg`!

## Conclusion

By combining TestNG Listeners with the Allure `@Attachment` annotation, we have completely automated our debugging process. 

Engineers no longer need to re-run failing tests locally to figure out what broke. All the evidence—screenshots and DOM trees—is captured in real-time and embedded directly into the report for instant root-cause analysis!

While Allure is incredibly powerful, it isn't the only reporting framework on the market. In our next tutorial, we will explore its biggest competitor: **Extent Reports**. We will learn how to build dynamic, single-page HTML dashboards entirely via Java code!
