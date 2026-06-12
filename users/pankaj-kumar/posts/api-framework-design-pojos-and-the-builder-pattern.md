---
title: API Framework Design: POJOs and the Builder Pattern
date: 20-Aug-2024
lastUpdated: 12-Jun-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [selenium, java, restassured, api-testing, hybrid-automation, framework, builder-pattern, lombok]
category: UI Automation
categories: [UI Automation, Selenium, Java]
excerpt: >-
  Design a highly modular API engine for your Hybrid Framework by eliminating raw JSON strings and encapsulating RestAssured logic using Lombok, POJOs, and the Builder Pattern.
readTime: 5 min read
---

# API Framework Design: POJOs and the Builder Pattern

In our previous articles, we explored how to combine Selenium and RestAssured into a Hybrid Framework and how to secure it using OpenAPI Contract Testing. 

However, as your hybrid framework scales, writing raw JSON strings and repetitive RestAssured `given().when().then()` chains inside your test classes quickly becomes unmaintainable. 

In this article, we will completely decouple our API logic from our test classes. By leveraging **POJOs (Plain Old Java Objects)**, **Lombok**, and the **Builder Pattern**, we will construct a highly reusable, clean, and modular API backend engine.

---

## 1. The Problem with Raw JSON

Writing JSON payloads as Java Strings is prone to syntax errors and is incredibly hard to maintain.

```java
// BAD PRACTICE: Hard-coded JSON string
String payload = "{ \"name\": \"John Doe\", \"email\": \"john@example.com\", \"role\": \"ADMIN\" }";
 
ApiClient.getBaseSpec()
    .body(payload)
    .post("/v1/users");
```

If the API requires an additional field tomorrow, you must manually hunt down and update every string payload in your framework.

---

## 2. Introducing POJOs and Lombok

Instead of strings, we map our API payloads to Java Objects (POJOs). To eliminate boilerplate code like getters, setters, and constructors, we use the **Lombok** library.

### Add Lombok to `pom.xml`:
```xml
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.18.30</version>
    <scope>provided</scope>
</dependency>
```

### Creating the User POJO
By annotating our class with `@Data` and `@Builder`, Lombok automatically generates everything we need behind the scenes.

```java
package com.mycodeyatra.api.models;
 
import lombok.Builder;
import lombok.Data;
 
@Data
@Builder
public class UserPayload {
    private String name;
    private String email;
    private String role;
}
```

---

## 3. The Power of the Builder Pattern

The `@Builder` annotation unlocks the Builder Pattern, allowing us to instantiate test data in a fluent, readable manner without massive constructors.

```java
// Creating dynamic test data using the Builder Pattern
UserPayload testUser = UserPayload.builder()
    .name("John Doe")
    .email("john.doe@automation.com")
    .role("ADMIN")
    .build();
```

When passed to RestAssured, the Jackson Databind library will automatically serialize this Java Object into a perfectly formatted JSON object.

---

## 4. Encapsulating Logic in Service Classes

Tests should only contain assertions and business flows. They should *not* contain HTTP routing logic. We solve this by creating Service Classes. 

A Service Class (e.g., `UserService`) acts as an adapter, hiding the RestAssured implementation from the test.

```java
package com.mycodeyatra.api.services;
 
import com.mycodeyatra.api.ApiClient;
import com.mycodeyatra.api.models.UserPayload;
import io.restassured.response.Response;
 
public class UserService {
 
    private static final String ENDPOINT = "/v1/users";
 
    public static Response createUser(UserPayload payload) {
        return ApiClient.getBaseSpec()
                .body(payload)
                .post(ENDPOINT);
    }
 
    public static Response getUser(String userId) {
        return ApiClient.getBaseSpec()
                .get(ENDPOINT + "/" + userId);
    }
 
    public static Response deleteUser(String userId) {
        return ApiClient.getBaseSpec()
                .delete(ENDPOINT + "/" + userId);
    }
}
```

---

## 5. The Final Hybrid Test

Now, let's look at how beautifully clean our Hybrid Test becomes. The test class has absolutely no knowledge of JSON strings, HTTP verbs, or REST endpoints.

```java
package com.mycodeyatra.tests;
 
import com.mycodeyatra.api.models.UserPayload;
import com.mycodeyatra.api.services.UserService;
import com.mycodeyatra.pages.AdminDashboardPage;
import io.restassured.response.Response;
import org.testng.Assert;
import org.testng.annotations.Test;
 
public class HybridAdminTest extends BaseTest {
 
    @Test
    public void verifyAdminCanViewCreatedUser() {
 
        // 1. Generate Data with Builder
        UserPayload newUser = UserPayload.builder()
                .name("Alice")
                .email("alice@automation.com")
                .role("CUSTOMER")
                .build();
 
        // 2. Setup via API Service
        Response apiResponse = UserService.createUser(newUser);
        Assert.assertEquals(apiResponse.getStatusCode(), 201);
        String userId = apiResponse.jsonPath().getString("id");
 
        // 3. Validate via UI
        AdminDashboardPage dashboard = new AdminDashboardPage(driver);
        dashboard.loginAsAdmin();
        dashboard.searchForUser(newUser.getEmail());
 
        Assert.assertTrue(dashboard.isUserPresent(newUser.getName()), 
                "User created via API was not found in Admin UI!");
 
        // 4. Teardown via API Service
        UserService.deleteUser(userId);
    }
}
```

---

## Architecture Overview

![diagram_1](https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/posts/api-framework-design-pojos-and-the-builder-pattern/images/diagram_1.png)

## Conclusion

By implementing POJOs, Lombok Builders, and isolated API Service Classes, your Hybrid Framework is now incredibly modular. If an endpoint changes, you update a single string in the `UserService`. If a JSON field changes, you update a single variable in the `UserPayload` POJO. 

This architecture drastically reduces maintenance overhead and makes your automated tests as clean and readable as plain English.
