---
title: Cryptographic Assertions: JWT Token Validation in Selenium Java
date: 19-Jun-2025
lastUpdated: 19-Jun-2025
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [java, selenium, devsecops, jwt, token-validation, security, local-storage, test-automation]
category: Selenium Java
categories: [Selenium Java, Security]
excerpt: >-
  Stop developers from accidentally exposing passwords in plain text. Learn how to extract JSON Web Tokens (JWT) from Local Storage via Selenium WebStorage and decode their cryptographic claims using Java jjwt.
readTime: 6 min read
---

# Cryptographic Assertions: JWT Token Validation in Selenium Java

In modern web architecture, most authentication is stateless. When a user logs in, the backend server does not create a session in a database. Instead, it generates a **JSON Web Token (JWT)**, cryptographically signs it, and sends it back to the browser. 

The browser stores this token (usually in Local Storage or a Cookie) and attaches it to every subsequent API request.

If this token is misconfigured—for example, if it lacks an expiration date, or if it exposes sensitive Personally Identifiable Information (PII) in plain text—the application is fundamentally insecure. 

In this tutorial, we will write a Selenium Java test that logs into an application, extracts the JWT from Local Storage, decodes it, and asserts its cryptographic claims!

---

## 1. Understanding the Anatomy of a JWT

A JWT is a long string separated by two periods into three parts: `Header.Payload.Signature`.

Example:
`eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiZXhwIjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c`

The middle part, the **Payload**, is simply Base64 encoded JSON. Anyone who captures the token can decode the payload and read the data inside. 

**This is why developers must NEVER put passwords or credit card numbers inside a JWT payload.** Our Selenium test is going to verify exactly that!

---

## 2. Extracting the Token from Local Storage

Selenium provides a very simple `LocalStorage` interface to interact with the browser's storage engine.

```java
import org.openqa.selenium.By;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.html5.LocalStorage;
import org.openqa.selenium.html5.WebStorage;
import org.testng.Assert;
import org.testng.annotations.Test;
public class TokenValidationTest {
    @Test
    public void testJwtSecurityClaims() {
        ChromeDriver driver = new ChromeDriver();
        // 1. Perform standard UI Login
        driver.get("https://practice.mycodeyatra.com/login");
        driver.findElement(By.id("username")).sendKeys("admin");
        driver.findElement(By.id("password")).sendKeys("secret");
        driver.findElement(By.id("login-btn")).click();
        // Wait for dashboard to load
        driver.findElement(By.id("dashboard-welcome"));
        // 2. Extract the JWT from Local Storage
        LocalStorage local = ((WebStorage) driver).getLocalStorage();
        String jwtToken = local.getItem("auth_token");
        Assert.assertNotNull(jwtToken, "JWT Token was not found in Local Storage!");
        // Now we need to decode and validate it!
        validateJwtClaims(jwtToken);
        driver.quit();
    }
}
```

---

## 3. Decoding and Asserting the JWT Claims

To decode and validate the JWT, we will use the popular `jjwt` (Java JWT) library. Add this to your `pom.xml`:

```xml
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-api</artifactId>
    <version>0.11.5</version>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-impl</artifactId>
    <version>0.11.5</version>
    <scope>runtime</scope>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-jackson</artifactId>
    <version>0.11.5</version>
    <scope>runtime</scope>
</dependency>
```

Now, let's write the `validateJwtClaims` method. We will decode the token WITHOUT verifying the signature (since we don't have the backend's private key), and we will assert that the developer followed security best practices.

```java
import io.jsonwebtoken.Claims;
import io.jsonwebtoken.Jwts;
private void validateJwtClaims(String jwtToken) {
    // 1. Parse the token payload (Base64 decode)
    // We strip the signature part to decode without a secret key
    String unsignedToken = jwtToken.substring(0, jwtToken.lastIndexOf('.') + 1);
    Claims claims = Jwts.parserBuilder()
            .build()
            .parseClaimsJwt(unsignedToken)
            .getBody();
    // 2. Assert Expiration Date exists and is not too long
    Assert.assertNotNull(claims.getExpiration(), "SECURITY FLAW: Token has no expiration date!");
    long timeToLive = claims.getExpiration().getTime() - System.currentTimeMillis();
    long maxAllowedTTL = 1000 * 60 * 60 * 24; // 24 hours
    Assert.assertTrue(timeToLive <= maxAllowedTTL, "SECURITY FLAW: Token lifespan exceeds 24 hours!");
    // 3. Assert PII (Personally Identifiable Information) is NOT exposed
    Assert.assertNull(claims.get("password"), "CRITICAL FLAW: Password exposed in plain text in JWT!");
    Assert.assertNull(claims.get("credit_card"), "CRITICAL FLAW: Financial data exposed in JWT!");
    // 4. Validate the Issuer (iss)
    Assert.assertEquals(claims.getIssuer(), "https://api.mycodeyatra.com", "Token generated by unknown issuer!");
    System.out.println("✅ JWT Security Claims Validated Successfully!");
}
```

---

## 4. Why This Matters

If you only test the UI, you will never know if the backend developer accidentally included the user's Social Security Number inside the JWT payload. 

By actively extracting the token from Selenium's `WebStorage` interface and decoding it in Java, you instantly catch major security flaws before they ever reach production. 

## Conclusion

Test Automation is evolving. By bridging the gap between Frontend UI testing and Backend Cryptography, QA Engineers can provide immense value to the security posture of their organization.

In our next tutorial, we will take this to the absolute extreme by integrating our Selenium framework directly with **OWASP ZAP** to execute automated Penetration Scans against our web applications!
