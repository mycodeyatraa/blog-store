---
title: Advanced Token Management: Injecting JWT Headers via CDP in Selenium 4
date: 28-Jul-2025
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [java, selenium, authentication, cdp, jwt, headers]
category: Authentication & Security
categories: [Authentication & Security, Java, Automation]
excerpt: >-
  Unlock the true power of Selenium 4! Learn how to use the Chrome DevTools Protocol (CDP) to intercept raw network traffic and dynamically inject Bearer JWT Tokens into the HTTP Authorization headers.
readTime: 6 min read
---

# Advanced Token Management: Injecting JWT Headers via CDP in Selenium 4

In modern highly-secure web applications, frontend frameworks (like React or Angular) do not always store authentication tokens in Cookies or LocalStorage. To protect against Cross-Site Scripting (XSS) attacks, some architectures hold the **JSON Web Token (JWT)** in memory and attach it dynamically to the HTTP `Authorization` headers on every outgoing network request.

If the token isn't in LocalStorage or Cookies, how can we inject it to bypass the UI login screen? 

In Selenium 4, we have direct access to the **Chrome DevTools Protocol (CDP)**. This allows us to intercept the browser's raw network traffic in real-time and inject custom HTTP Headers before the requests ever reach the server!

---

## 1. Understanding the Chrome DevTools Protocol (CDP)

Historically, Selenium was a "black box" that only interacted with the DOM (clicking buttons, typing text). It had no control over the underlying network layer.

With the release of Selenium 4, WebDriver now implements a bi-directional communication channel with Chromium-based browsers (Chrome, Edge). This allows us to use the `NetworkInterception` API to read, block, or modify HTTP Requests on the fly!

---

## 2. Injecting Authorization Headers

Let's write a test where we fetch a valid JWT token via a fast backend API call (using RestAssured or standard Java HTTP clients), and then we intercept *every single outgoing request* from the Chrome browser to permanently attach that token to the `Authorization: Bearer <token>` header.

**CdpHeaderInjectionTest.java**

```java
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.devtools.NetworkInterceptor;
import org.openqa.selenium.remote.http.Filter;
import org.openqa.selenium.remote.http.HttpRequest;
import org.openqa.selenium.remote.http.HttpResponse;
import org.testng.Assert;
import org.testng.annotations.Test;
public class CdpHeaderInjectionTest {
    @Test
    public void testTokenInjectionViaNetworkInterceptor() {
        // 1. Fetch JWT via fast API call (Simulated)
        String myJwtToken = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdW...";
        WebDriver driver = new ChromeDriver();
        // 2. Create a Network Interception Filter
        Filter headerInjector = next -> (HttpRequest request) -> {
            // Append the Authorization header to EVERY request leaving the browser
            request.addHeader("Authorization", "Bearer " + myJwtToken);
            // Proceed with the modified request
            HttpResponse response = next.execute(request);
            return response;
        };
        // 3. Attach the Interceptor to the Driver via CDP
        try (NetworkInterceptor interceptor = new NetworkInterceptor(driver, headerInjector)) {
            // 4. Navigate directly to the protected dashboard!
            // When the browser makes this request, our interceptor will dynamically 
            // attach the Bearer token to the HTTP Headers, granting us instant access!
            driver.get("https://mycodeyatra.com/secure-dashboard");
            Assert.assertTrue(driver.getPageSource().contains("Secure Data Accessed"));
        } // The interceptor automatically closes at the end of this block
        driver.quit();
    }
}
```

### How This Works
1. When you call `driver.get()`, Chrome attempts to send an HTTP GET request to the server.
2. The `NetworkInterceptor` pauses the request instantly.
3. The `Filter` lambda intercepts the raw `HttpRequest` object and uses `request.addHeader()` to inject the JWT.
4. The modified request is released and sent to the server.
5. The server sees a valid Bearer token and returns the authenticated Dashboard page instead of redirecting to the Login screen!

---

## 3. Targeted Interception

Injecting a token into *every* request (including requests for images, CSS, or third-party analytics scripts) is inefficient and can sometimes cause errors if third-party servers reject unexpected Authorization headers.

We can update our Filter to only inject the token for API requests heading to our specific backend domain!

```java
Filter targetedInjector = next -> (HttpRequest request) -> {
    // Only inject the header if the request is hitting OUR backend API
    if (request.getUri().contains("api.mycodeyatra.com")) {
        request.addHeader("Authorization", "Bearer " + myJwtToken);
    }
    return next.execute(request);
};
```

## Conclusion

By mastering the Chrome DevTools Protocol (CDP) in Selenium 4, you unlock superpowers that were previously impossible in UI automation.
- You are no longer restricted to just the DOM.
- If your application hides authentication tokens in memory and relies on HTTP headers, you can easily bypass the UI by fetching the token via API and using `NetworkInterceptor` to attach it to the `Authorization` header on the fly.

### 🎉 The Java Authentication Mastery Series is Complete!
Congratulations! Over this 4-part series, you have conquered the hardest security mechanisms in the industry: Cookies, LocalStorage, SSO, Multi-Factor Authentication, Session Serialization, Concurrency, and CDP Network Interception. 

You are now a true Selenium Security Architect!
