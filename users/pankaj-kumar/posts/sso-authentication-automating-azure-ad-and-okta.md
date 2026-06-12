---
title: SSO Authentication: Automating Azure AD and Okta
date: 30-Aug-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [selenium, java, sso, authentication, azure-ad, okta]
category: UI Automation
categories: [UI Automation, Selenium, Java]
excerpt: >-
  Master the complexities of automating SSO flows. Learn how to handle cross-domain redirects and write robust Selenium scripts to automate Azure AD and Okta logins.
readTime: 5 min read
---

# SSO Authentication: Automating Azure AD and Okta

Modern enterprise applications rarely rely on simple username/password forms stored in their own databases. Instead, they delegate authentication to identity providers (IdPs) like Microsoft Azure Active Directory (Azure AD), Okta, or PingIdentity via Single Sign-On (SSO).

Automating SSO flows with Selenium is notoriously tricky. Your script leaves your application's domain, navigates through external vendor screens, handles complex HTTP redirects, and eventually lands back on your application.

In this article, we will dissect how to reliably automate SSO logins, specifically focusing on Microsoft Azure AD, and explore strategies for making these tests robust.

---

## 1. The Challenge of SSO Automation

When a user clicks "Log in with Microsoft" on your site, the following happens:
1. The browser redirects to `login.microsoftonline.com`.
2. The user enters their email.
3. Microsoft processes the email and redirects to a password screen (sometimes a custom corporate portal).
4. The user enters their password.
5. Microsoft asks "Stay signed in?".
6. Microsoft issues an OAuth token and redirects *back* to your application.

This multi-step, multi-domain flow completely breaks standard Selenium `Implicit Waits` and static Page Object Models.

---

## 2. Automating Azure AD Login

To automate this, we need a dedicated Page Object just for the Microsoft login flow. We must rely heavily on **Explicit Waits** (`WebDriverWait`) because Microsoft's screens load dynamically using React/Angular, meaning the DOM is constantly shifting.

Here is a robust implementation for an Azure AD login flow:

```java
package com.mycodeyatra.pages;
 
import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.support.ui.ExpectedConditions;
import org.openqa.selenium.support.ui.WebDriverWait;
import java.time.Duration;
 
public class MicrosoftSSOPage {
 
    private WebDriver driver;
    private WebDriverWait wait;
 
    // Microsoft Login Locators
    private By emailInput = By.name("loginfmt");
    private By nextButton = By.id("idSIButton9");
    private By passwordInput = By.name("passwd");
    private By signInButton = By.id("idSIButton9"); // Reused ID by Microsoft
    private By staySignedInNoBtn = By.id("idBtn_Back");
 
    public MicrosoftSSOPage(WebDriver driver) {
        this.driver = driver;
        this.wait = new WebDriverWait(driver, Duration.ofSeconds(15));
    }
 
    public void loginToMicrosoft(String email, String password) {
        // 1. Enter Email
        wait.until(ExpectedConditions.visibilityOfElementLocated(emailInput)).sendKeys(email);
        driver.findElement(nextButton).click();
 
        // 2. Enter Password (Crucial to wait for it to become visible)
        wait.until(ExpectedConditions.visibilityOfElementLocated(passwordInput)).sendKeys(password);
 
        // Wait for the button text to change or become clickable
        wait.until(ExpectedConditions.elementToBeClickable(signInButton)).click();
 
        // 3. Handle the "Stay signed in?" prompt
        try {
            wait.until(ExpectedConditions.elementToBeClickable(staySignedInNoBtn)).click();
        } catch (Exception e) {
            // Sometimes Microsoft skips this screen depending on tenant settings
            System.out.println("Stay Signed In prompt did not appear, continuing...");
        }
    }
}
```

---

## 3. Integrating SSO into Your Test

Now, let's look at how your actual test uses this SSO Page Object. 

```java
@Test
public void verifySsoLogin() {
    // 1. Navigate to your app
    driver.get("https://mycodeyatra.com");
 
    // 2. Click the SSO button which redirects to Microsoft
    driver.findElement(By.id("azure-sso-btn")).click();
 
    // 3. Pass control to the Microsoft SSO Page Object
    MicrosoftSSOPage ssoPage = new MicrosoftSSOPage(driver);
    ssoPage.loginToMicrosoft("automation@mycompany.com", "SecurePass123!");
 
    // 4. Wait for the redirect back to your application
    WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
    wait.until(ExpectedConditions.urlContains("mycodeyatra.com/dashboard"));
 
    // 5. Verify Successful Login
    Assert.assertTrue(driver.getTitle().contains("Dashboard"));
}
```

---

## 4. The "Conditional MFA" Problem

The code above works perfectly—until your IT department enforces Multi-Factor Authentication (MFA). If Microsoft suddenly prompts your Selenium script for a push notification or an SMS code, the test will instantly fail.

**How do we solve this?**
1. **Service Accounts:** Work with your IT/Security team to provision dedicated "Service Accounts" (e.g., `test_user_1@company.com`) that have MFA explicitly disabled via Azure Conditional Access policies.
2. **IP Whitelisting:** Some companies allow bypassing MFA if the login originates from the corporate VPN or a specific CI/CD IP address (like your Jenkins server).

*(We will cover how to automate TOTP Authenticator apps in a future article if IT refuses to disable MFA!)*

---

## System Architecture

Here is the exact network flow during an automated SSO test:

![diagram_1](https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/posts/sso-authentication-automating-azure-ad-and-okta/images/diagram_1.png)

## Conclusion

Automating SSO flows like Azure AD or Okta requires a deep understanding of Explicit Waits and complex network redirects. By isolating the external login logic into its own dedicated Page Object, you keep your tests clean and maintainable.

However, as we discussed, MFA is the ultimate enemy of SSO automation. In our next article, we will dive into **MFA Concepts** and learn how to use Java to generate real-time TOTP (Google Authenticator) codes directly inside your Selenium script!
