---
title: API Authentication and Authorization
date: 2025-02-06
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [restassured, java, security, authentication, jwt]
category: API Testing
categories: [API Testing, Automation, Java]
excerpt: >-
  Learn how to securely automate API endpoints by handling Basic Auth and Bearer Tokens using RestAssured's built-in authentication mechanisms.
readTime: 6 min read
---

In modern API development, very few endpoints are completely public. The vast majority of operations require the user to prove who they are (Authentication) and prove they have the right to perform the action (Authorization).

In this tutorial, we will learn how to handle the two most common types of API security mechanisms using RestAssured: **Basic Authentication** and **Bearer Tokens (JWT)**.

---

## 1. Basic Authentication

Basic Authentication is a simple, built-in HTTP mechanism where the client sends the `username` and `password` encoded in Base64 directly in the `Authorization` header.

Instead of manually encoding these strings, RestAssured provides an `.auth().basic()` method that handles everything seamlessly!

```java
@Test(priority = 1)
public void testBasicAuthentication() {
    System.out.println("\\n--- Executing Basic Auth Test ---");
    // Using built-in basic auth configuration
    Response response = RestAssured.given()
            .auth()
            .basic("admin", "admin")
            .when()
            .get("/api/auth/basic")
            .then()
            .extract().response();
    Assert.assertEquals(response.getStatusCode(), 200);
    System.out.println("Basic Auth Response: " + response.jsonPath().getString("message"));
}
```

## 2. Bearer Tokens (JWT)

Modern applications generally avoid sending passwords on every request. Instead, they require you to log in once via a `/login` endpoint. The server returns a **Token** (often a JSON Web Token), and you pass that token in the `Authorization: Bearer <token>` header for all subsequent requests.

RestAssured supports this via the `.auth().oauth2(token)` method! Let's write a two-part test: first, we log in to extract the token, and then we use it to access a protected profile route.

```java
@Test(priority = 2)
public void testBearerTokenAuthentication() {
    System.out.println("\\n--- Executing Bearer Token Login ---");
    // 1. Authenticate and extract token
    Map<String, String> loginPayload = new HashMap<>();
    loginPayload.put("email", "admin@example.com");
    loginPayload.put("password", "password123");
    Response loginResponse = RestAssured.given()
            .contentType(ContentType.JSON)
            .body(loginPayload)
            .when()
            .post("/api/auth/login")
            .then()
            .extract().response();
    Assert.assertEquals(loginResponse.getStatusCode(), 200);
    String token = loginResponse.jsonPath().getString("token");
    System.out.println("Extracted Token: " + token.substring(0, 15) + "...");
    System.out.println("\\n--- Accessing Protected Route with Token ---");
    // 2. Use the token to access a protected route
    Response profileResponse = RestAssured.given()
            .auth()
            .oauth2(token) // RestAssured uses oauth2() for Bearer tokens
            .when()
            .get("/api/auth/profile")
            .then()
            .extract().response();
    Assert.assertEquals(profileResponse.getStatusCode(), 200);
    System.out.println("Profile Response: " + profileResponse.jsonPath().getString("message"));
}
```

---

## The Execution Output

When we run these tests against our MyCodeYatra Mock API Server (`mvn clean test`), here is the exact console output showing successful authentication:

```text
--- Executing Basic Auth Test ---
Basic Auth Response: Basic Auth successful
--- Executing Bearer Token Login ---
Extracted Token: eyJhbGciOiJIUzI...
--- Accessing Protected Route with Token ---
Profile Response: Welcome to your protected profile!
[INFO] Tests run: 2, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 13.41 s -- in TestSuite
```

By mastering these built-in authentication handlers, your automation scripts can safely interact with any secured backend system!

In the next tutorial, we will learn how to parse and extract data from XML responses!
