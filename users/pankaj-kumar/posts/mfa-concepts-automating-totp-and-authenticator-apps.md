---
title: MFA Concepts: Automating TOTP and Authenticator Apps
date: 02-Sep-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [selenium, java, mfa, totp, authentication, framework]
category: UI Automation
categories: [UI Automation, Selenium, Java]
excerpt: >-
  Bypass the ultimate testing roadblock. Learn how to automatically generate 6-digit MFA codes directly in Java using TOTP algorithms for your Selenium scripts.
readTime: 5 min read
---

# MFA Concepts: Automating TOTP and Authenticator Apps

In the previous article, we managed to successfully automate Single Sign-On (SSO) logins using Microsoft Azure AD. However, we noted a massive roadblock: **Multi-Factor Authentication (MFA)**. 

If your application prompts you for a 6-digit code from Google Authenticator, Authy, or Microsoft Authenticator, standard Selenium scripts come to a grinding halt. You cannot physically pull out your phone and type the code during an automated CI/CD pipeline run.

In this article, we will learn how to bypass this limitation by mathematically generating our own Time-Based One-Time Passwords (TOTP) directly in Java!

---

## 1. How Authenticator Apps Actually Work

When you first set up MFA on a website, it usually shows you a QR code. 
That QR code contains a hidden string known as a **Secret Key** (e.g., `JBSWY3DPEHPK3PXP`). 

When you scan it, your phone saves this key. Every 30 seconds, your phone takes the Secret Key, combines it with the current Unix timestamp, and runs it through an HMAC-SHA1 cryptographic algorithm to generate a 6-digit code.

Because the algorithm is completely open-source (RFC 6238), we don't actually need a physical phone. We just need the Secret Key and a Java library!

---

## 2. Setting Up the TOTP Library

To generate the codes in Java, we will use the `aerogear-otp-java` library. Add the following dependency to your `pom.xml`:

```xml
<dependency>
    <groupId>org.jboss.aerogear</groupId>
    <artifactId>aerogear-otp-java</artifactId>
    <version>1.0.0</version>
</dependency>
```

---

## 3. Extracting the Secret Key

Before you can automate the test, you need the Secret Key. 

1. Go to the application and initiate the MFA setup for your dedicated test user account.
2. When the QR code appears, **do not scan it**. Instead, look for a button that says "Trouble scanning?" or "Enter code manually". 
3. The site will display the raw Secret Key string (e.g., `JBSWY3DPEHPK3PXP`).
4. Save this Secret Key securely in your test framework (e.g., in an environment variable or a secure properties file).

---

## 4. Generating the Code in Java

Now we can write a simple utility method that generates the 6-digit code on demand.

```java
package com.mycodeyatra.utils;
 
import org.jboss.aerogear.security.otp.Totp;
 
public class MfaUtility {
 
    /**
     * Generates a 6-digit TOTP code based on the provided secret key.
     */
    public static String getTwoFactorCode(String secretKey) {
        Totp totp = new Totp(secretKey);
        return totp.now();
    }
}
```

---

## 5. Integrating with Selenium

Let's plug this directly into an automated login flow. 

When the UI asks for the 6-digit code, our script instantly generates the correct code and injects it into the input field!

```java
package com.mycodeyatra.tests;
 
import com.mycodeyatra.utils.MfaUtility;
import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.support.ui.ExpectedConditions;
import org.openqa.selenium.support.ui.WebDriverWait;
import org.testng.Assert;
import org.testng.annotations.Test;
import java.time.Duration;
 
public class MfaLoginTest extends BaseTest {
 
    // Store this in your .env or secure vault!
    private static final String MFA_SECRET = "JBSWY3DPEHPK3PXP"; 
 
    @Test
    public void verifyMfaLoginFlow() {
 
        // 1. Enter standard credentials
        driver.get("https://mycodeyatra.com/login");
        driver.findElement(By.id("username")).sendKeys("mfa_user@mycompany.com");
        driver.findElement(By.id("password")).sendKeys("SecurePass123!");
        driver.findElement(By.id("loginBtn")).click();
 
        // 2. Wait for the MFA challenge screen
        WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
        wait.until(ExpectedConditions.visibilityOfElementLocated(By.id("mfaCodeInput")));
 
        // 3. Generate the TOTP code dynamically in Java
        String generatedCode = MfaUtility.getTwoFactorCode(MFA_SECRET);
        System.out.println("Injecting MFA Code: " + generatedCode);
 
        // 4. Inject the code and submit
        driver.findElement(By.id("mfaCodeInput")).sendKeys(generatedCode);
        driver.findElement(By.id("submitMfaBtn")).click();
 
        // 5. Verify successful authentication
        wait.until(ExpectedConditions.urlContains("/dashboard"));
        Assert.assertTrue(driver.getTitle().contains("Dashboard"));
    }
}
```

---

## System Architecture

Here is how the automated MFA validation process looks architecturally:

![diagram_1](https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/posts/mfa-concepts-automating-totp-and-authenticator-apps/images/diagram_1.png)

## Considerations

While this method works perfectly for TOTP (Authenticator Apps), it **does not work for SMS or Email MFA**. 

If your company uses SMS for MFA, you must route those messages to a virtual phone number (like Twilio) and use RestAssured to fetch the code via Twilio's API. For Email MFA, you would use an IMAP library in Java or a service like Mailtrap to read the incoming emails.

In our next article, we will wrap up our authentication journey by exploring **Token Management**—learning how to extract JWTs and API keys straight from the browser's local storage to use in our API tests!
