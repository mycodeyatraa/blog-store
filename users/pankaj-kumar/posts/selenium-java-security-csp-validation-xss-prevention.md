---
title: Penetration Testing with Selenium: Validating Content Security Policies (CSP)
date: 17-Jun-2026
lastUpdated: 19-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [java, selenium, devsecops, csp, security, xss, test-automation]
category: Selenium Java
categories: [Selenium Java, Security]
excerpt: >-
  Stop XSS attacks in their tracks. Learn how to write Selenium Java tests that passively intercept CSP headers via DevTools, and actively attempt inline script injections to verify your security defenses.
readTime: 6 min read
---

# Penetration Testing with Selenium: Validating Content Security Policies (CSP)

In our previous tutorial, we learned how to use Selenium 4's Chrome DevTools Protocol (CDP) to intercept network traffic and validate basic HTTP security headers. 

Today, we are going to tackle the undisputed king of web security: the **Content Security Policy (CSP)**.

A misconfigured CSP is the primary cause of Cross-Site Scripting (XSS) attacks. If an attacker manages to inject a malicious JavaScript tag into your application, a properly configured CSP will literally block the browser from executing it. In this tutorial, we will learn how to write Selenium Java tests to ensure your CSP is bulletproof!

---

## 1. What is a Content Security Policy?

A CSP is an HTTP response header that acts as a strict "Allowlist" for the browser. It tells the browser exactly which domains are allowed to load scripts, images, styles, and fonts on the current page.

Here is an example of a highly secure CSP header:
`Content-Security-Policy: default-src 'self'; script-src 'self' https://trusted-cdn.com; object-src 'none';`

**What this means:**
- `default-src 'self'`: By default, only load assets that come from the exact same domain as the website.
- `script-src 'self' https://trusted-cdn.com`: Only execute JavaScript that comes from our own server or `trusted-cdn.com`. **If an attacker injects a `<script src="http://evil-hacker.com/steal.js">` tag into a comment box, the browser will refuse to execute it!**
- `object-src 'none'`: Completely ban legacy Flash or Java applets.

---

## 2. Reading the CSP Header via Selenium

Just like we did with HSTS and X-Frame-Options, we can use the CDP `Network.responseReceived` event to extract the CSP header from the backend server.

```java
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.devtools.DevTools;
import org.openqa.selenium.devtools.v120.network.Network;
import org.openqa.selenium.devtools.v120.network.model.Headers;
import org.testng.Assert;
import org.testng.annotations.Test;
import java.util.Optional;
public class CspValidationTest {
    @Test
    public void validateCspHeader() {
        ChromeDriver driver = new ChromeDriver();
        DevTools devTools = driver.getDevTools();
        devTools.createSession();
        devTools.send(Network.enable(Optional.empty(), Optional.empty(), Optional.empty()));
        // We use a StringBuilder because we are modifying the value inside a lambda expression
        StringBuilder cspHeader = new StringBuilder();
        devTools.addListener(Network.responseReceived(), response -> {
            if (response.getResponse().getUrl().equals("https://practice.mycodeyatra.com/")) {
                Headers headers = response.getResponse().getHeaders();
                String csp = (String) headers.get("content-security-policy");
                if (csp != null) {
                    cspHeader.append(csp);
                }
            }
        });
        driver.get("https://practice.mycodeyatra.com/");
        // 1. Verify the header actually exists!
        Assert.assertTrue(cspHeader.length() > 0, "CRITICAL VULNERABILITY: No CSP Header found!");
        String policy = cspHeader.toString();
        // 2. Verify we are not allowing 'unsafe-inline' scripts!
        // 'unsafe-inline' allows attackers to inject <script>alert(1)</script> directly into the HTML!
        Assert.assertFalse(policy.contains("'unsafe-inline'"), 
            "SECURITY FLAW: CSP allows 'unsafe-inline' script execution!");
        // 3. Verify object-src is disabled to prevent Flash injection
        Assert.assertTrue(policy.contains("object-src 'none'"), 
            "SECURITY FLAW: Legacy object embeds are not blocked!");
        driver.quit();
    }
}
```

---

## 3. Advanced Active Injection Testing

Reading the header is great, but we are QA Engineers! We don't just read configuration files—we actively try to break the application!

We can use Selenium to actively inject a script tag into the DOM and verify that the browser's CSP engine physically blocks it from executing.

```java
import org.openqa.selenium.JavascriptExecutor;
import org.openqa.selenium.WebDriverException;
@Test
public void testCspActivelyBlocksInlineScripts() {
    ChromeDriver driver = new ChromeDriver();
    driver.get("https://practice.mycodeyatra.com/");
    JavascriptExecutor js = (JavascriptExecutor) driver;
    try {
        // Attempt to execute an inline script that changes the background to red
        // If the CSP is secure (no 'unsafe-inline'), the browser will block this and throw an error!
        js.executeScript(
            "var script = document.createElement('script');" +
            "script.innerText = \"document.body.style.backgroundColor = 'red';\";" +
            "document.head.appendChild(script);"
        );
        // If we reach this line, the script executed successfully. The CSP is broken!
        Assert.fail("VULNERABILITY: The browser allowed our injected inline script to execute!");
    } catch (WebDriverException e) {
        // This is exactly what we want! 
        // The browser's CSP engine intercepted the script and threw an EvalError/SecurityError.
        System.out.println("✅ Security Test Passed! The browser actively blocked the inline script injection.");
    } finally {
        driver.quit();
    }
}
```

## Conclusion

By combining passive header analysis via CDP with active JavaScript injection via `JavascriptExecutor`, your Selenium test suite transforms into an automated Penetration Testing tool! You can now confidently guarantee that your frontend is immune to the most common forms of Cross-Site Scripting.

In our next security tutorial, we will explore **Token Validation**, learning how to securely extract and assert the cryptographic claims of JWTs stored inside the browser's Local Storage!
