---
title: Hybrid Automation: Combining Selenium UI and RestAssured API
date: 15-Aug-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [selenium, java, restassured, api-testing, hybrid-automation, framework]
category: UI Automation
categories: [UI Automation, Selenium, Java]
excerpt: >-
  Skip the basics and dive straight into Hybrid Automation. Learn how to combine Selenium WebDriver and RestAssured in a single test to achieve unmatched speed and reliability.
readTime: 5 min read
---

# Hybrid Automation: Combining Selenium UI and RestAssured API

When building an enterprise test automation framework, relying solely on UI automation is a trap. UI tests are notoriously slow and prone to flakiness. On the other hand, API tests are lightning fast but cannot guarantee that the user interface renders correctly. 

Since we have already covered the depths of standalone API testing in our previous RestAssured series, we will skip the basics. Instead, we are diving straight into **Hybrid Automation**—the architectural pattern of merging Selenium WebDriver and RestAssured in the exact same test to achieve maximum speed and reliability.

---

## Why Combine UI and API Automation?

A traditional E2E UI test for "Editing a User Profile" looks like this:
1. UI: Login
2. UI: Navigate to User Creation Page
3. UI: Fill out form and Create User
4. UI: Navigate to Profile
5. UI: Edit Profile
6. UI: Delete User (Teardown)

This is incredibly slow. By using a Hybrid approach, we bypass the UI for setup and teardown:

1. **API:** Authenticate and get Token (100ms)
2. **API:** Create User (100ms)
3. **UI:** Login and Edit Profile (3 seconds)
4. **API:** Delete User (100ms)

You save immense execution time while still validating the critical UI components.

---

## 1. Setting up the Hybrid Architecture

We need both Selenium and RestAssured available in our test execution context. Since we already have a robust Spring DI and `ThreadLocal` setup for our UI, we will inject a base `RequestSpecification` for our API calls.

```java
package com.mycodeyatra.api;
 
import io.restassured.RestAssured;
import io.restassured.specification.RequestSpecification;
 
public class ApiClient {
 
    public static RequestSpecification getBaseSpec() {
        return RestAssured.given()
                .baseUri("https://api.mycodeyatra.com")
                .header("Content-Type", "application/json")
                .header("Accept", "application/json");
    }
}
```

---

## 2. Use Case 1: API Test Data Setup + UI Validation

In this scenario, we use RestAssured to instantly create a product in our backend, and then we use Selenium to verify that the product properly renders on the frontend storefront.

```java
package com.mycodeyatra.tests;
 
import com.mycodeyatra.api.ApiClient;
import com.mycodeyatra.pages.StoreFrontPage;
import io.restassured.response.Response;
import org.testng.Assert;
import org.testng.annotations.Test;
 
public class HybridProductTest extends BaseTest {
 
    @Test
    public void verifyProductRendersOnStorefront() {
        // 1. Setup Data via API (Lightning Fast)
        String productPayload = "{ \"name\": \"Selenium Masterclass\", \"price\": 99.99 }";
 
        Response apiResponse = ApiClient.getBaseSpec()
                .body(productPayload)
                .post("/v1/products");
 
        Assert.assertEquals(apiResponse.getStatusCode(), 201);
        String productId = apiResponse.jsonPath().getString("id");
 
        // 2. Validate via UI (Selenium)
        StoreFrontPage storePage = new StoreFrontPage(driver);
        storePage.navigateToStore();
        storePage.searchFor("Selenium Masterclass");
 
        Assert.assertTrue(storePage.isProductVisible(productId), 
                "Product created via API was not found on the UI!");
 
        // 3. Teardown via API
        ApiClient.getBaseSpec().delete("/v1/products/" + productId);
    }
}
```

---

## 3. Use Case 2: UI Action + API State Validation

Sometimes you perform an action on the UI (like checking out a shopping cart), and you want to verify that the backend database state updated correctly without navigating through 5 different admin screens.

```java
package com.mycodeyatra.tests;
 
import com.mycodeyatra.api.ApiClient;
import com.mycodeyatra.pages.CheckoutPage;
import org.testng.Assert;
import org.testng.annotations.Test;
 
public class HybridCheckoutTest extends BaseTest {
 
    @Test
    public void verifyUiCheckoutUpdatesBackend() {
        // 1. Perform complex UI action
        CheckoutPage checkout = new CheckoutPage(driver);
        checkout.navigateToCart();
        String orderId = checkout.completePurchase("Visa", "1234-5678-9012-3456");
 
        // 2. Instantly verify backend state via API
        String orderStatus = ApiClient.getBaseSpec()
                .get("/v1/orders/" + orderId)
                .jsonPath().getString("status");
 
        Assert.assertEquals(orderStatus, "PAID", 
                "Backend order status did not update after UI checkout!");
    }
}
```

---

## System Workflow

![diagram_1](https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/posts/hybrid-automation-combining-selenium-ui-and-restassured-api/images/diagram_1.png)

## The Power of Hybrid Automation

By bridging the gap between Selenium and RestAssured, you achieve:
1. **Unmatched Speed:** Test data setup and teardown drops from minutes to milliseconds.
2. **Reliability:** Removing UI navigation steps reduces the chance of flaky element locators failing your tests.
3. **True E2E Coverage:** You validate exactly what the user sees, while strictly ensuring the backend contracts are upheld.

In our next article, we will integrate **API Contract Testing with OpenAPI** directly into this hybrid architecture to automatically catch breaking schema changes!
