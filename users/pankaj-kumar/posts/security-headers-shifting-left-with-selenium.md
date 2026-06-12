---
title: Security Headers: Shifting Left with Selenium
date: 08-Sep-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [selenium, java, security-testing, headers, cdp, restassured]
category: UI Automation
categories: [UI Automation, Selenium, Java]
excerpt: >-
  Automate security testing directly in your framework. Learn how to validate HTTP security headers using RestAssured and Selenium 4's Chrome DevTools Protocol.
readTime: 5 min read
---

# Security Headers: Shifting Left with Selenium

Traditionally, security testing is an afterthought, performed manually by a specialized InfoSec team just days before a major release. This often leads to delayed launches and frantic, last-minute hotfixes.

In Phase 7, **Advanced Security Testing**, we are going to explore how SDETs can "shift-left" by writing automated security checks directly into the existing Selenium Java framework. 

We begin with the easiest, yet most critical, line of defense for any web application: **HTTP Security Headers**.

---

## 1. What Are Security Headers?

When a browser requests a page from your application, the server responds with the HTML content and a set of HTTP headers. Security Headers tell the browser how to behave to prevent malicious attacks.

The "Big Four" headers you should always assert are:
- `Strict-Transport-Security` (HSTS): Forces all traffic over HTTPS, preventing downgrade attacks.
- `X-Frame-Options`: Prevents Clickjacking by disallowing your site from being embedded in an iframe.
- `X-Content-Type-Options`: Prevents MIME-sniffing vulnerabilities.
- `Content-Security-Policy` (CSP): Mitigates Cross-Site Scripting (XSS) by restricting where scripts can be loaded from.

---

## 2. Validating Headers using RestAssured

Since headers are part of the raw HTTP response, the absolute fastest way to test them is by utilizing your API testing layer. 

If you built the Hybrid Framework we discussed in Phase 5, you already have RestAssured installed! We can write a blazing-fast unit test that asserts the presence of these headers on the application's root URL.

```java
package com.mycodeyatra.security;
 
import io.restassured.RestAssured;
import io.restassured.response.Response;
import org.testng.Assert;
import org.testng.annotations.Test;
 
public class SecurityHeaderApiTest {
 
    @Test
    public void verifySecurityHeadersArePresent() {
        // Hit the front-end URL
        Response response = RestAssured.get("https://mycodeyatra.com");
 
        // Assert HTTP 200 OK
        Assert.assertEquals(response.statusCode(), 200);
 
        // Assert the Big Four Headers
        Assert.assertNotNull(response.header("Strict-Transport-Security"), "HSTS Header Missing!");
        Assert.assertEquals(response.header("X-Frame-Options"), "DENY", "X-Frame-Options should be DENY!");
        Assert.assertEquals(response.header("X-Content-Type-Options"), "nosniff", "MIME Sniffing protection missing!");
        Assert.assertNotNull(response.header("Content-Security-Policy"), "CSP Header Missing!");
    }
}
```
*Execution Time: ~400 milliseconds.*

---

## 3. Validating Headers using Selenium 4 (CDP)

What if you don't want to use an API library? What if you want your Selenium UI test to automatically intercept the network traffic and validate the headers *while* the browser is navigating the page?

Thanks to **Selenium 4**, we can access the Chrome DevTools Protocol (CDP) directly from Java to intercept responses!

```java
package com.mycodeyatra.security;
 
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.devtools.DevTools;
import org.openqa.selenium.devtools.v114.network.Network;
import org.testng.Assert;
import org.testng.annotations.AfterMethod;
import org.testng.annotations.BeforeMethod;
import org.testng.annotations.Test;
 
import java.util.Optional;
 
public class SecurityHeaderUITest {
 
    private ChromeDriver driver;
 
    @BeforeMethod
    public void setup() {
        driver = new ChromeDriver();
    }
 
    @Test
    public void verifyHeadersDuringNavigation() {
        // 1. Get the DevTools session
        DevTools devTools = driver.getDevTools();
        devTools.createSession();
 
        // 2. Enable Network tracking
        devTools.send(Network.enable(Optional.empty(), Optional.empty(), Optional.empty()));
 
        // 3. Add a listener to intercept all responses
        devTools.addListener(Network.responseReceived(), responseReceived -> {
            String url = responseReceived.getResponse().getUrl();
 
            // Only check headers for our main application domain, ignore 3rd party scripts
            if (url.equals("https://mycodeyatra.com/")) {
                var headers = responseReceived.getResponse().getHeaders();
 
                System.out.println("Validating Headers for: " + url);
                Assert.assertTrue(headers.containsKey("Strict-Transport-Security"), "Missing HSTS");
                Assert.assertEquals(headers.get("x-frame-options").toString().toUpperCase(), "DENY");
            }
        });
 
        // 4. Navigate (The listener will instantly catch the response)
        driver.get("https://mycodeyatra.com");
    }
 
    @AfterMethod
    public void teardown() {
        driver.quit();
    }
}
```

---

## Architectural Comparison

![diagram_1](https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/posts/security-headers-shifting-left-with-selenium/images/diagram_1.png)

## Which approach should you choose?

If you just need to check the homepage, **RestAssured is better**. It is significantly faster and doesn't require spinning up a physical Chrome executable. 

However, if your application has dynamic routes where security headers might be accidentally stripped off by a load balancer on specific pages (like `/checkout`), using **Selenium 4 CDP** is vastly superior. You can leave the network listener running in the background for your entire UI test suite, acting as a passive security scanner!

In our next article, we will take a deep dive into the most complex security header of all: **Content Security Policy (CSP)**, and learn how to validate it.
