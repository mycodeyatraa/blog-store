---
title: Advanced Session Handling: Reusing WebDriver Sessions
date: 25-Aug-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [selenium, java, session-handling, authentication, debugging, chrome-devtools]
category: UI Automation
categories: [UI Automation, Selenium, Java]
excerpt: >-
  Learn how WebDriver Session IDs work and how to hijack an active Chrome instance using Remote Debugging Mode to drastically speed up test debugging.
readTime: 5 min read
---

# Advanced Session Handling: Reusing WebDriver Sessions

In our previous article on Cookie Management, we explored how to save an authenticated state to a file and inject it into fresh browser instances. While that drastically speeds up test execution, it still requires opening a brand new browser window for every test.

What if you don't want to open a new browser? What if you are debugging a complex test script, and instead of restarting the entire flow from scratch, you want Selenium to simply "attach" to the Chrome window that is already open on your screen?

Welcome to **Advanced Session Handling**. In this article, we cover how WebDriver Session IDs work and how to hijack an active browser session.

---

## 1. What is a WebDriver Session ID?

When you initialize a new `ChromeDriver`, the Selenium language bindings send a `POST /session` HTTP request to the browser driver. The driver launches the physical browser and returns a unique **Session ID**.

Every subsequent command you issue (like `findElement` or `click`) is sent with this Session ID so the driver knows which specific browser window to control.

```java
WebDriver driver = new ChromeDriver();
SessionId sessionId = ((RemoteWebDriver) driver).getSessionId();
System.out.println("Current Session ID: " + sessionId.toString());
```

Once `driver.quit()` is called, the driver terminates the browser and destroys the Session ID.

---

## 2. The Use Case: Debugging Flaky Tests

Imagine a 20-step E2E test that fails on Step 19. 
If you fix the code, you have to run the test again from Step 1, waiting 3 minutes just to see if your fix on Step 19 actually worked. 

If we could tell our IDE to execute *only* Step 19 and attach to the browser window that just failed, we would save hours of debugging time. We can achieve this by launching Chrome in **Remote Debugging Mode**.

---

## 3. Starting Chrome in Debugging Mode

Before running your Selenium script, you must start an instance of Chrome directly from your command line (or terminal) and tell it to open a debugging port.

**On Windows:**
```cmd
chrome.exe --remote-debugging-port=9222 --user-data-dir="C:\ChromeProfile"
```

**On Mac:**
```bash
/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome --remote-debugging-port=9222 --user-data-dir="/tmp/ChromeProfile"
```

This will open a physical Chrome window. You can manually navigate to your application and log in.

---

## 4. Hijacking the Session with Selenium

Now that we have a browser running with an open debugging port, we can configure our `ChromeOptions` in Java to connect to this exact instance instead of launching a new one.

```java
package com.mycodeyatra.tests;
 
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.chrome.ChromeOptions;
import org.testng.annotations.Test;
 
public class SessionHijackTest {
 
    @Test
    public void attachToExistingBrowser() {
 
        ChromeOptions options = new ChromeOptions();
 
        // Tell Selenium to connect to the browser we opened manually
        options.setExperimentalOption("debuggerAddress", "127.0.0.1:9222");
 
        // Initialize driver with these options
        WebDriver driver = new ChromeDriver(options);
 
        // Selenium will NOT open a new window. It will take control of the existing one!
        System.out.println("Hijacked Page Title: " + driver.getTitle());
 
        // Now you can test just the step you are debugging
        // driver.findElement(By.id("step19-button")).click();
    }
}
```

---

## System Architecture

Here is exactly what is happening under the hood when we hijack a debugging port:

![diagram_1](https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/posts/advanced-session-handling-reusing-webdriver-sessions/images/diagram_1.png)

## Why This Matters for Enterprise Frameworks

1. **Massive Debugging Speed:** You no longer need to wait for 20 setup steps to execute just to test one locator fix.
2. **Bypassing MFA:** If your application uses 2FA (like an SMS code) that is impossible to automate, you can manually log in via the debugging browser, and then point your test suite to that browser to run the rest of the regression pack.
3. **Persistent State:** Everything stored in the browser (LocalStorage, IndexedDB, SessionStorage) remains completely intact.

In our next article, we will tackle **Multi-User Testing**, where we will run an Admin session and a Customer session simultaneously in the exact same test!
