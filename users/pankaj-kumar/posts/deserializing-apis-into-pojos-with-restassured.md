---
title: Deserializing APIs into POJOs
date: 12-Feb-2025
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [restassured, java, pojo, lombok, deserialization]
category: API Testing
categories: [API Testing, Automation, Java]
excerpt: >-
  Learn how to eliminate raw JSON string parsing by automatically mapping HTTP API responses directly into strongly-typed Java objects using POJOs and Lombok.
readTime: 6 min read
---

While validating raw JSON fields using `jsonPath()` is excellent for quick scripts, Enterprise API Automation Frameworks demand strong typing. When your API payload has 100 properties, parsing them manually is an absolute nightmare.

The solution? **POJOs (Plain Old Java Objects)**. In this tutorial, we will learn how to seamlessly serialize and deserialize API payloads into robust Java objects!

---

## 1. Creating the POJO Model

First, we define a Java class that perfectly mirrors the structure of our JSON payload. To eliminate boilerplate code (like getters, setters, and constructors), we will use the **Lombok** library!

*(Ensure `lombok` and `jackson-databind` are in your `pom.xml`)*

```java
package com.mycodeyatra.models;
import lombok.Data;
import lombok.NoArgsConstructor;
import lombok.AllArgsConstructor;
@Data
@NoArgsConstructor
@AllArgsConstructor
public class User {
    private String id;
    private String name;
    private String email;
    private String role;
    private String createdAt;
}
```

That's it! `@Data` automatically generates everything we need behind the scenes.

## 2. Deserializing Responses directly into POJOs

Now let's write a RestAssured test. We will create a `User` object, pass it into the request body (Serialization), and then fetch it back and cast the HTTP Response directly into the `User` class (Deserialization)!

```java
@BeforeClass
public void setup() {
    RestAssured.baseURI = "http://localhost:8080";
    RestAssured.basePath = "/api";
    // Setup: Create a user to deserialize later (Serialization)
    User setupUser = new User();
    setupUser.setName("POJO Tester");
    setupUser.setEmail("pojo@example.com");
    setupUser.setRole("user");
    Response response = RestAssured.given()
            .contentType(ContentType.JSON)
            .body(setupUser) // RestAssured automatically converts this POJO to JSON!
            .when()
            .post("/users")
            .then()
            .extract().response();
    createdUserId = response.jsonPath().getString("id");
}
@Test
public void testDeserializationToPojo() {
    System.out.println("\\n--- Executing POJO Deserialization Test ---");
    // Execute GET request and directly deserialize the JSON response!
    User fetchedUser = RestAssured.given()
            .when()
            .get("/users/" + createdUserId)
            .then()
            .statusCode(200)
            .extract()
            .as(User.class); // Magic happens here!
    // Validate using strong typing
    System.out.println("Deserialized User ID: " + fetchedUser.getId());
    System.out.println("Deserialized User Name: " + fetchedUser.getName());
    Assert.assertEquals(fetchedUser.getId(), createdUserId);
    Assert.assertEquals(fetchedUser.getName(), "POJO Tester");
    Assert.assertEquals(fetchedUser.getEmail(), "pojo@example.com");
    Assert.assertNotNull(fetchedUser.getCreatedAt());
}
```

## The Execution Output

When you run this test (`mvn clean test`), RestAssured effortlessly maps the network payload bytes straight into your compiled Java code:

```text
--- Executing POJO Deserialization Test ---
Deserialized User ID: d912b3...
Deserialized User Name: POJO Tester
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 13.91 s -- in TestSuite
```

By leveraging `extract().as(Class.class)`, you unlock the full power of Java's type safety, your IDE's autocomplete, and you eliminate the risk of typos in string-based `jsonPath()` queries!

In our upcoming tutorials, we will tie the whole framework together with CI/CD integration.
