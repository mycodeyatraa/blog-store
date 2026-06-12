---
title: Token Management: Extracting JWTs from the Browser
date: 05-Sep-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [selenium, java, jwt, authentication, restassured, framework, localstorage]
category: UI Automation
categories: [UI Automation, Selenium, Java]
excerpt: >-
  Bridge your UI and API tests. Learn how to extract JWTs from LocalStorage and SessionStorage using JavascriptExecutor to pass authentication tokens to RestAssured.
readTime: 5 min read
---

# Token Management: Extracting JWTs from the Browser

Throughout our **Advanced Authentication Mastery** series, we have covered how to bypass UI logins using cookies, how to handle SSO, and how to conquer MFA using TOTP algorithms. 

But what if you are writing a true **Hybrid Framework** (like we designed in Phase 5) where you want to use the UI to log in once, and then use APIs to generate all your test data instantly?

To do that, your API automation library (RestAssured) needs the authentication token. In modern Single Page Applications (SPAs) built with React, Angular, or Vue, these tokens are rarely stored in traditional Cookies. Instead, they are stored in the browser's **Local Storage** or **Session Storage** as JSON Web Tokens (JWTs).

In this article, we will learn how to extract these tokens directly from the browser using Selenium and pass them to our API layer!

---

## 1. Understanding Web Storage

Unlike cookies, which are automatically sent to the server with every HTTP request, **Local Storage** and **Session Storage** are client-side storage mechanisms accessed purely via JavaScript. 

- **LocalStorage:** Persists even after the browser is closed.
- **SessionStorage:** Cleared when the page session ends (when the tab is closed).

When a user logs into a modern SPA, the backend returns a JWT, and the frontend JavaScript saves it:
```javascript
localStorage.setItem("auth_token", "eyJhbGciOiJIUzI1NiIsIn...");
```

---

## 2. Extracting Tokens with JavascriptExecutor

Because `LocalStorage` is purely a JavaScript concept, Selenium's standard `driver.manage()` API cannot access it. Instead, we must cast our WebDriver to a `JavascriptExecutor` and execute raw JS in the browser context.

Here is a utility class that allows us to effortlessly read and write to Web Storage:

```java
package com.mycodeyatra.utils;
 
import org.openqa.selenium.JavascriptExecutor;
import org.openqa.selenium.WebDriver;
 
public class WebStorageUtils {
 
    private JavascriptExecutor js;
 
    public WebStorageUtils(WebDriver driver) {
        this.js = (JavascriptExecutor) driver;
    }
 
    // --- Local Storage Methods --- //
 
    public String getLocalStorageItem(String key) {
        return (String) js.executeScript(String.format("return window.localStorage.getItem('%s');", key));
    }
 
    public void setLocalStorageItem(String key, String value) {
        js.executeScript(String.format("window.localStorage.setItem('%s','%s');", key, value));
    }
 
    // --- Session Storage Methods --- //
 
    public String getSessionStorageItem(String key) {
        return (String) js.executeScript(String.format("return window.sessionStorage.getItem('%s');", key));
    }
}
```

---

## 3. The Ultimate Hybrid Workflow

Now let's see this in action. We are going to write a test that:
1. Logs into the application via the UI.
2. Extracts the `access_token` from Local Storage.
3. Passes that token to RestAssured to instantly create an order via the backend API.
4. Refreshes the UI to verify the order appears.

```java
package com.mycodeyatra.tests;
 
import com.mycodeyatra.utils.WebStorageUtils;
import io.restassured.RestAssured;
import org.openqa.selenium.By;
import org.testng.Assert;
import org.testng.annotations.Test;
 
public class HybridTokenTest extends BaseTest {
 
    @Test
    public void createOrderViaApiAfterUiLogin() {
 
        // 1. Standard UI Login
        driver.get("https://mycodeyatra.com/login");
        driver.findElement(By.id("username")).sendKeys("hybrid_user");
        driver.findElement(By.id("password")).sendKeys("Pass123!");
        driver.findElement(By.id("loginBtn")).click();
 
        // Wait for dashboard to load (ensures token is stored)
        wait.until(ExpectedConditions.urlContains("/dashboard"));
 
        // 2. Extract the JWT from Local Storage
        WebStorageUtils storageUtils = new WebStorageUtils(driver);
        String jwtToken = storageUtils.getLocalStorageItem("access_token");
 
        System.out.println("Extracted Token: " + jwtToken);
 
        // 3. Pass Token to RestAssured to create an order instantly
        String apiEndpoint = "https://api.mycodeyatra.com/v1/orders";
        String orderPayload = "{ \"item\": \"Laptop\", \"qty\": 1 }";
 
        RestAssured.given()
            .header("Authorization", "Bearer " + jwtToken)
            .header("Content-Type", "application/json")
            .body(orderPayload)
        .when()
            .post(apiEndpoint)
        .then()
            .statusCode(201); // Created
 
        // 4. Refresh the UI and verify the order is visible
        driver.navigate().refresh();
        boolean isOrderVisible = driver.findElement(By.xpath("//td[text()='Laptop']")).isDisplayed();
 
        Assert.assertTrue(isOrderVisible, "The API-created order did not appear in the UI!");
    }
}
```

---

## System Architecture

Here is the exact flow of data between your Browser, your Selenium Script, and the API Backend:

![diagram_1](https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/posts/token-management-extracting-jwts-from-the-browser/images/diagram_1.png)

## Conclusion

Extracting tokens directly from the browser's storage is the ultimate "bridge" for a Hybrid Automation Framework. It allows you to utilize the UI to handle complex authentication flows (like SSO or MFA) while still unlocking the blistering speed of API automation for test data setup and teardown.

This officially concludes **Phase 6: Advanced Authentication Mastery**! You now possess the knowledge to automate Cookies, Session IDs, Multi-User roles, SSO, MFA, and JWT Tokens.

In our next Phase, we will shift our focus back to the UI and dive deep into Advanced UI Interactions (Shadow DOM, Canvas, and complex Actions). Get ready!
