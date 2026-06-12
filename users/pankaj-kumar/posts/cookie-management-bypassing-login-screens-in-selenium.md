---
title: Cookie Management: Bypassing Login Screens in Selenium
date: 22-Aug-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [selenium, java, cookies, authentication, performance, framework]
category: UI Automation
categories: [UI Automation, Selenium, Java]
excerpt: >-
  Learn how to manage cookies directly through Selenium WebDriver to save sessions and completely bypass slow UI login screens.
readTime: 5 min read
---

# Cookie Management: Bypassing Login Screens in Selenium

One of the most time-consuming parts of automated UI testing is the login process. If you have 100 tests and each one starts by navigating to the login page, entering credentials, handling a captcha, and waiting for the dashboard to load, you are wasting an immense amount of execution time.

In Phase 6, **Advanced Authentication Mastery**, we will explore how to drastically speed up and secure your test executions. We start with the foundation of web authentication: **Cookies**. 

By managing cookies directly through Selenium WebDriver, we can log in *once*, save the session, and instantly inject it into all subsequent tests, completely bypassing the login UI.

---

## 1. How Cookies Work in Selenium

When you log into a web application, the server responds with a session token (often a JSESSIONID or a JWT) stored as a cookie in your browser. As long as this cookie is present and unexpired, the server considers you authenticated.

Selenium provides a robust `Options` interface to interact with these cookies:

```java
// Get all cookies
Set<Cookie> cookies = driver.manage().getCookies();
 
// Get a specific cookie by name
Cookie sessionCookie = driver.manage().getCookieNamed("JSESSIONID");
 
// Delete a specific cookie (simulating a logout)
driver.manage().deleteCookieNamed("JSESSIONID");
 
// Delete all cookies (clearing session state)
driver.manage().deleteAllCookies();
```

---

## 2. The Slow Way: Logging in Every Time

A standard test script logs in through the UI for every single test:

```java
@Test
public void slowTest() {
    driver.get("https://mycodeyatra.com/login");
    driver.findElement(By.id("username")).sendKeys("admin");
    driver.findElement(By.id("password")).sendKeys("Password123!");
    driver.findElement(By.id("loginBtn")).click();
 
    // Now perform the actual test logic...
}
```
If the login takes 5 seconds, running 100 tests adds **8.3 minutes** of pure login overhead.

---

## 3. The Fast Way: Injecting Cookies

If we know the authentication cookie format, we can navigate directly to the application domain, inject the cookie, and then jump straight to the secured page.

```java
@Test
public void fastTestWithCookieInjection() {
    // 1. Navigate to the domain first (Required by Selenium before adding cookies)
    driver.get("https://mycodeyatra.com");
 
    // 2. Create the authentication cookie
    Cookie authCookie = new Cookie.Builder("session_token", "abc123xyz890")
            .domain("mycodeyatra.com")
            .path("/")
            .isSecure(true)
            .build();
 
    // 3. Inject the cookie into the browser
    driver.manage().addCookie(authCookie);
 
    // 4. Navigate directly to the secured dashboard (Bypassing login)
    driver.get("https://mycodeyatra.com/admin/dashboard");
 
    // 5. Verify we are authenticated
    Assert.assertTrue(driver.getTitle().contains("Dashboard"));
}
```

---

## 4. Persisting Sessions Across Tests

Hardcoding cookie values isn't practical because tokens expire. The ultimate pattern is to:
1. Have a `SetupTest` log in via the UI once.
2. Extract the resulting cookies and save them to a file (like a JSON or properties file).
3. Have all other tests read that file and inject the cookies.

### Step A: Saving the Cookie
```java
public void saveSession() {
    // Assume driver is already logged in
    Set<Cookie> cookies = driver.manage().getCookies();
 
    // Write cookies to a file (using standard Java I/O or Jackson)
    File file = new File("target/session.data");
    try (BufferedWriter writer = new BufferedWriter(new FileWriter(file))) {
        for (Cookie c : cookies) {
            writer.write(c.getName() + ";" + c.getValue() + ";" + c.getDomain() + ";" + 
                         c.getPath() + ";" + c.getExpiry() + ";" + c.isSecure());
            writer.newLine();
        }
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```

### Step B: Loading the Cookie in Subsequent Tests
```java
public void loadSession() {
    driver.get("https://mycodeyatra.com"); // Must hit domain first
 
    File file = new File("target/session.data");
    try (BufferedReader reader = new BufferedReader(new FileReader(file))) {
        String line;
        while ((line = reader.readLine()) != null) {
            String[] parts = line.split(";");
            Cookie cookie = new Cookie.Builder(parts[0], parts[1])
                    .domain(parts[2])
                    .path(parts[3])
                    .isSecure(Boolean.parseBoolean(parts[5]))
                    .build();
            driver.manage().addCookie(cookie);
        }
    } catch (Exception e) {
        e.printStackTrace();
    }
 
    // Navigate to secured area!
    driver.get("https://mycodeyatra.com/admin");
}
```

---

## Architecture Overview

![diagram_1](https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/posts/cookie-management-bypassing-login-screens-in-selenium/images/diagram_1.png)

## Conclusion

By managing cookies efficiently, you drastically reduce test execution time and avoid flaky UI login elements. However, modern applications don't just rely on standard cookies—they use complex Session IDs, Local Storage, and OAuth tokens. 

In our next article, we will dive deeper into **Session Handling** and how to gracefully reuse the exact same WebDriver session ID to avoid opening multiple browser instances altogether!
