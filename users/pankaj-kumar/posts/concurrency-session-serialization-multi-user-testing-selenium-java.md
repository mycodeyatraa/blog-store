---
title: Concurrency and Session Serialization in Selenium Java
date: 25-Jul-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [java, selenium, authentication, serialization, concurrency, multi-user]
category: Authentication & Security
categories: [Authentication & Security, Java, Automation]
excerpt: >-
  Take your test framework to the next level! Learn how to serialize browser states to physical files, and run multiple Incognito WebDrivers in a single test case to simulate real-time chat and multi-user collaboration.
readTime: 6 min read
---

# Concurrency and Session Serialization in Selenium Java

In our previous articles, we learned how to inject Cookies and LocalStorage tokens to bypass the UI login screen. But what if you have an application that requires extreme performance, or an application where multiple users need to interact with each other in real-time?

In this article, we will tackle two of the most advanced authentication patterns in UI Automation:
1. **Session Serialization**: Saving a logged-in browser state to a physical file so it can be reused across different test suites days later.
2. **Multi-User Concurrency Testing**: Running two different WebDriver instances in the exact same test case to simulate real-time collaboration.

---

## 1. Session Serialization (Saving State to Disk)

Imagine you have a complex MFA or SSO login flow that takes 15 seconds to complete. Even if you use an API to bypass the UI, the API itself might be heavily rate-limited.

To solve this, we can perform the expensive login process **exactly once**. We then serialize (export) all of the browser's Cookies into a JSON or physical text file. For the next 24 hours, every other test can simply read that text file and inject the cookies instantly!

### Step 1: Exporting Cookies to a File

**SessionSerializer.java**

```java
import org.openqa.selenium.Cookie;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import java.io.BufferedWriter;
import java.io.File;
import java.io.FileWriter;
import java.io.IOException;
public class SessionSerializer {
    public void loginAndSaveSession() {
        WebDriver driver = new ChromeDriver();
        driver.get("https://mycodeyatra.com/login");
        // ... perform expensive UI login or API login here ...
        // Export Cookies to a file
        File file = new File("auth_session.data");
        try (BufferedWriter writer = new BufferedWriter(new FileWriter(file))) {
            for (Cookie cookie : driver.manage().getCookies()) {
                writer.write(cookie.getName() + ";" + 
                             cookie.getValue() + ";" + 
                             cookie.getDomain() + ";" + 
                             cookie.getPath() + ";" + 
                             cookie.getExpiry() + ";" + 
                             cookie.isSecure());
                writer.newLine();
            }
            System.out.println("Session saved to auth_session.data");
        } catch (IOException e) {
            e.printStackTrace();
        }
        driver.quit();
    }
}
```

### Step 2: Restoring the Session

Now, in your Pytest or TestNG `@BeforeSuite`, you can read this file and instantly inject the state!

**SessionDeserializer.java**

```java
import org.openqa.selenium.Cookie;
import org.openqa.selenium.WebDriver;
import java.io.BufferedReader;
import java.io.File;
import java.io.FileReader;
import java.util.Date;
import java.util.StringTokenizer;
public class SessionDeserializer {
    public void injectSession(WebDriver driver) {
        driver.get("https://mycodeyatra.com"); // Must navigate to domain first!
        File file = new File("auth_session.data");
        try (BufferedReader reader = new BufferedReader(new FileReader(file))) {
            String line;
            while ((line = reader.readLine()) != null) {
                StringTokenizer token = new StringTokenizer(line, ";");
                while (token.hasMoreTokens()) {
                    String name = token.nextToken();
                    String value = token.nextToken();
                    String domain = token.nextToken();
                    String path = token.nextToken();
                    Date expiry = null; // Parse date string if needed
                    boolean isSecure = Boolean.parseBoolean(token.nextToken());
                    Cookie cookie = new Cookie(name, value, domain, path, expiry, isSecure);
                    driver.manage().addCookie(cookie);
                }
            }
            System.out.println("Session restored successfully!");
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

---

## 2. Multi-User Concurrency Testing

Most QA Engineers only test from the perspective of a single user. But what if you are testing a **Live Chat Application**, a **Google Docs-style collaborative editor**, or an **Admin/Customer support portal**?

You need to test what happens when User A sends a message and verify that User B receives it instantly.

To do this, we must instantiate **Two WebDrivers** in the exact same test method! 

### The Golden Rule of Multi-User Testing
Browsers share state (Cookies/LocalStorage) *per user profile*. If you launch two standard Chrome instances, logging into Driver A might accidentally log Driver B in as well! 
To prevent this collision, we must run both drivers in **Incognito/Private mode**, or point them to separate temporary profile directories.

**MultiUserChatTest.java**

```java
import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.chrome.ChromeOptions;
import org.testng.Assert;
import org.testng.annotations.Test;
public class MultiUserChatTest {
    @Test
    public void testLiveChatBetweenTwoUsers() throws InterruptedException {
        // 1. Setup Driver A (The Customer) in Incognito
        ChromeOptions optionsA = new ChromeOptions();
        optionsA.addArguments("--incognito");
        WebDriver customerDriver = new ChromeDriver(optionsA);
        // 2. Setup Driver B (The Admin) in Incognito
        ChromeOptions optionsB = new ChromeOptions();
        optionsB.addArguments("--incognito");
        WebDriver adminDriver = new ChromeDriver(optionsB);
        try {
            // 3. Login Customer
            customerDriver.get("https://mycodeyatra.com/chat");
            customerDriver.manage().addCookie(new Cookie("JSESSIONID", "customer_token_123"));
            customerDriver.navigate().refresh();
            // 4. Login Admin
            adminDriver.get("https://mycodeyatra.com/admin/chat");
            adminDriver.manage().addCookie(new Cookie("JSESSIONID", "admin_token_999"));
            adminDriver.navigate().refresh();
            // 5. Customer sends a message
            customerDriver.findElement(By.id("message-box")).sendKeys("Hello, I need help!");
            customerDriver.findElement(By.id("send-btn")).click();
            // 6. Admin verifies the message arrives via WebSockets in real-time
            Thread.sleep(1000); // Wait for WebSocket delivery
            String incomingMessage = adminDriver.findElement(By.id("incoming-msg")).getText();
            Assert.assertEquals(incomingMessage, "Hello, I need help!");
            // 7. Admin replies... (You get the idea!)
        } finally {
            // 8. Always close BOTH drivers!
            customerDriver.quit();
            adminDriver.quit();
        }
    }
}
```

## Conclusion

By mastering these advanced patterns, you elevate your framework from a simple script runner to an Enterprise Validation engine:
- Use **Session Serialization** to save authenticated states to `.data` or `.json` files, completely eliminating redundant logins across multiple test suites.
- Use **Multi-User Concurrency** with multiple Incognito WebDrivers to test real-time WebSockets, chat applications, and complex multi-role workflows.

In our final article for the Java Authentication Mastery series, we will tackle **Token Management**, teaching you how to inject JWTs directly into Network Headers using the Chrome DevTools Protocol (CDP)!
