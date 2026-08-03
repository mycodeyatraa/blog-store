---
title: Data-Driven Testing with JUnit 5 & REST-Assured
date: 20-Feb-2026
lastUpdated: 20-Feb-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [restassured, java, junit5, data-driven, parameterized]
category: REST-Assured
categories: [REST-Assured, API Testing, Java]
excerpt: >-
  Run scalable parameterized API tests! Harness JUnit 5 @ParameterizedTest, @CsvSource, and @MethodSource with REST-Assured.
readTime: 8 min read
---

# Data-Driven Testing with JUnit 5 & REST-Assured

Testing an API endpoint against multiple sets of data (valid inputs, boundary conditions, invalid payloads) by copying and pasting test methods creates massive code duplication.

**Data-Driven Testing (DDT)** decouples test logic from test data. Combining **JUnit 5 parameterized tests** with **REST-Assured** enables executing a single test method across hundreds of data combinations cleanly.

---

## 1. JUnit 5 Parameterized Annotations

- **`@ValueSource`**: Feeds single primitive arguments.
- **`@CsvSource`**: Inlines comma-separated rows of test data.
- **`@CsvFileSource`**: Reads test data directly from external CSV files.
- **`@MethodSource`**: Feeds complex data structures (POJOs, Streams) from factory methods.

---

## 2. Executing Data-Driven API Test Suites

Below is a complete test suite demonstrating `@CsvSource` and `@MethodSource` with REST-Assured:

**src/test/java/com/mycodeyatra/DataDrivenApiTest.java**

```java
package com.mycodeyatra;
 
import io.restassured.RestAssured;
import io.restassured.http.ContentType;
import org.junit.jupiter.api.BeforeAll;
import org.junit.jupiter.params.ParameterizedTest;
import org.junit.jupiter.params.provider.CsvSource;
import org.junit.jupiter.params.provider.MethodSource;
 
import java.util.stream.Stream;
 
import static io.restassured.RestAssured.given;
import static org.hamcrest.Matchers.*;
 
public class DataDrivenApiTest {
 
    @BeforeAll
    public static void setup() {
        RestAssured.baseURI = "https://jsonplaceholder.typicode.com";
    }
 
    // 1. Inlined CSV Parameterized Test
    @ParameterizedTest(name = "Validate User ID {0} has username ''{1}''")
    @CsvSource({
        "1, Bret",
        "2, Antonette",
        "3, Samantha",
        "4, Karianne"
    })
    public void testUserUsernamesFromCsv(int userId, String expectedUsername) {
        given()
            .contentType(ContentType.JSON)
        .when()
            .get("/users/" + userId)
        .then()
            .statusCode(200)
            .body("username", equalTo(expectedUsername));
    }
 
    // 2. Complex Method Source Data Provider
    static Stream<UserData> userDataProvider() {
        return Stream.of(
            new UserData(1, "Leanne Graham", "Sincere@april.biz"),
            new UserData(2, "Ervin Howell", "Shanna@melissa.tv")
        );
    }
 
    @ParameterizedTest
    @MethodSource("userDataProvider")
    public void testUserDetailsFromMethodSource(UserData user) {
        given()
            .contentType(ContentType.JSON)
        .when()
            .get("/users/" + user.id)
        .then()
            .statusCode(200)
            .body("name", equalTo(user.name))
            .body("email", equalTo(user.email));
    }
 
    // Inner Data Model
    static class UserData {
        int id;
        String name;
        String email;
 
        UserData(int id, String name, String email) {
            this.id = id;
            this.name = name;
            this.email = email;
        }
    }
}
```

---

## Best Practices

1. **Clear Display Names**: Customize `@ParameterizedTest(name = "...")` to output readable test iteration names in CI reports.
2. **Separate Edge Cases**: Test positive flows and 400 Bad Request error cases using separate parameterized test methods.
