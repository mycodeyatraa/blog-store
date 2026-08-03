---
title: Validating Responses with Hamcrest Matchers in REST-Assured
date: 17-Feb-2026
lastUpdated: 17-Feb-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: ["restassured", "java", "hamcrest", "assertions", "matchers"]
category: REST-Assured
categories: ["REST-Assured", "API Testing", "Java"]
excerpt: >-
  Write fluent, human-readable API assertions! Master Hamcrest matchers for complex JSON response validation in REST-Assured.
readTime: 8 min read
---

# Validating Responses with Hamcrest Matchers in REST-Assured

Validating API responses goes far beyond checking `200 OK` status codes. Production test suites must assert nested JSON object fields, array item sizes, numerical ranges, and string patterns.

**Hamcrest Matchers** are natively integrated into REST-Assured's `.then()` fluent validation block, enabling clean declarative assertions.

---

## 1. Core Hamcrest Matchers Overview

Hamcrest provides a comprehensive suite of static matchers:

- **Core Matchers**: `equalTo()`, `is()`, `not()`, `nullValue()`, `notNullValue()`.
- **Collection Matchers**: `hasSize()`, `hasItems()`, `contains()`, `empty()`.
- **String Matchers**: `equalToIgnoringCase()`, `containsString()`, `startsWith()`, `matchesPattern()`.
- **Number Matchers**: `greaterThan()`, `lessThanOrEqualTo()`, `closeTo()`.

---

## 2. Validating Complex JSON Response Payloads

Below is an enterprise test class demonstrating Hamcrest assertions on complex nested API responses:

**src/test/java/com/mycodeyatra/HamcrestValidationTest.java**

```java
package com.mycodeyatra;
 
import io.restassured.RestAssured;
import io.restassured.http.ContentType;
import org.junit.jupiter.api.BeforeAll;
import org.junit.jupiter.api.Test;
 
import static io.restassured.RestAssured.given;
import static org.hamcrest.Matchers.*;
 
public class HamcrestValidationTest {
 
    @BeforeAll
    public static void setup() {
        RestAssured.baseURI = "https://jsonplaceholder.typicode.com";
    }
 
    @Test
    public void testComplexJsonValidations() {
        given()
            .contentType(ContentType.JSON)
        .when()
            .get("/users/1")
        .then()
            .statusCode(200)
            .header("Content-Type", containsString("application/json"))
            // Field Value Validation
            .body("name", equalTo("Leanne Graham"))
            .body("username", is(not(emptyOrNullString())))
            // Nested JSON Object Path Validation
            .body("address.city", equalTo("Gwenborough"))
            .body("address.geo.lat", notNullValue())
            // Company details
            .body("company.name", containsString("Romaguera"));
    }
 
    @Test
    public void testJsonArrayValidations() {
        given()
            .contentType(ContentType.JSON)
        .when()
            .get("/posts")
        .then()
            .statusCode(200)
            .body("$", hasSize(greaterThan(50)))
            .body("id", hasItems(1, 2, 3))
            .body("title", everyItem(notNullValue()));
    }
}
```

---

## 3. Best Practices for Response Assertions

1. **Chain Body Matchers**: Group multiple `body()` checks within a single `.then()` specification block to fail fast.
2. **Validate Headers & Response Times**: Include `.time(lessThan(2000L))` to assert SLA limits on response latency.
