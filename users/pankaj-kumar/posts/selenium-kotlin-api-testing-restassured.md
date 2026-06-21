---
title: "Integrating API Testing with RestAssured in Selenium Kotlin"
date: "2025-03-07"
description: "Expand your automation framework's capabilities by integrating RestAssured with Kotlin and Kotest to perform robust API validations alongside your UI tests."
tags: ["Selenium", "Kotlin", "RestAssured", "API Testing", "Kotest", "Automation"]
---

As your automation framework matures into an enterprise-grade solution, purely testing the UI is no longer sufficient. Modern applications rely heavily on backend microservices. To ensure end-to-end stability, your framework must be capable of verifying database states and API responses.

In Java, **RestAssured** is the undisputed champion of API testing. Fortunately, because Kotlin has flawless interoperability with Java, we can bring RestAssured directly into our Kotest framework!

In this 30th post of our Selenium Kotlin Mastery Series, we will integrate RestAssured and write our first API test.

### Step 1: Adding RestAssured to `pom.xml`

We first need to add the RestAssured dependency to our project. Open your `pom.xml` and add the following inside your `<dependencies>` block:

```xml
<dependency>
    <groupId>io.rest-assured</groupId>
    <artifactId>rest-assured</artifactId>
    <version>5.4.0</version>
    <scope>test</scope>
</dependency>
```

### Step 2: Writing the API Test in Kotest

RestAssured uses a fluent Builder Pattern featuring `given()`, `when()`, and `then()`. This perfectly complements our Kotest BDD structures!

Because `when` is a reserved keyword in Kotlin, we must surround it with backticks: `` `when`() ``. Let's hit a public mock API (`jsonplaceholder.typicode.com`) to demonstrate this integration.

```kotlin
package com.mycodeyatra.tests
import io.kotest.core.spec.style.StringSpec
import io.kotest.matchers.string.shouldContain
import io.restassured.RestAssured.given
import io.restassured.http.ContentType
import org.hamcrest.Matchers.equalTo
class Blog30_ApiTest : StringSpec({
    "Verify User API returns HTTP 200 and Correct Name" {
        // 1. Perform the API Request using RestAssured
        val response = given()
            .baseUri("https://jsonplaceholder.typicode.com")
            .contentType(ContentType.JSON)
        .`when`() // Escaped because 'when' is a Kotlin keyword
            .get("/users/1")
        .then()
            .statusCode(200) // Validate status code using Hamcrest
            .body("name", equalTo("Leanne Graham")) // Validate JSON payload
            .extract().response()
        // 2. We can seamlessly switch back to Kotest assertions!
        val responseBody = response.asString()
        responseBody shouldContain "Sincere@april.biz"
        println("API Response Successfully Validated: \n$responseBody")
    }
})
```

### Why Blend UI and API Testing?

By integrating RestAssured into your Selenium project, you unlock immense power:
1. **Test Data Setup:** Instead of slowly clicking through the UI to create a user for a test, you can instantly hit a `POST /users` API to seed your test data in 200 milliseconds.
2. **State Teardown:** Use a `DELETE /users` API call in your Kotest `afterSpec` block to cleanly erase your test data, ensuring environment stability.
3. **Hybrid Validation:** You can verify that a value displayed in the UI perfectly matches the JSON response emitted by the backend.

### Execution Output

```
[INFO] Running com.mycodeyatra.tests.Blog30_ApiTest
API Response Successfully Validated: 
{
  "id": 1,
  "name": "Leanne Graham",
  "username": "Bret",
  "email": "Sincere@april.biz",
...
Tests: 1, Passed: 1, Failed: 0
```

### Conclusion

Integrating RestAssured into our Kotlin framework requires exactly zero boilerplate. The fluent syntax translates beautifully, allowing us to build lightning-fast Hybrid Automation Frameworks!

In our next blog, we will explore **Visual Regression Testing in Kotlin**, learning how to do pixel-perfect image comparisons of our Web UI!
