---
title: JSON Parsing with Jackson & Gson in REST-Assured
date: 16-Feb-2026
lastUpdated: 16-Feb-2026
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: ["restassured", "java", "jackson", "gson", "json", "serialization"]
category: REST-Assured
categories: ["REST-Assured", "API Testing", "Java"]
excerpt: >-
  Master payload serialization and deserialization! Learn how to integrate Jackson and Gson with REST-Assured for seamless POJO mapping.
readTime: 8 min read
---

# JSON Parsing with Jackson & Gson in REST-Assured

In modern REST API test automation, constructing raw JSON string payloads manually is error-prone and hard to maintain. **Object Mapping** (Serialization & Deserialization) allows engineers to convert Java Plain Old Java Objects (**POJOs**) into JSON payloads and parse HTTP JSON responses back into strongly-typed Java objects automatically.

REST-Assured natively supports enterprise JSON object mappers like **Jackson** and **Gson**.

---

## 1. Core Concepts: Serialization vs. Deserialization

- **Serialization**: Converting a Java Object (e.g., `UserRequest`) into a JSON string sent in the HTTP Request Body.
- **Deserialization**: Converting the HTTP Response JSON string into a Java Object (e.g., `UserResponse`) for strong type-safe assertions.

```
 +------------------+   Serialization (Jackson/Gson)   +-------------------+
 |  Java POJO       | -------------------------------> |  JSON Payload     |
 |  (UserRequest)   |                                  |  {"name": "John"} |
 +------------------+                                  +-------------------+
          ^                                                      |
          |           Deserialization (Response Parsing)         |
          +------------------------------------------------------+
```

---

## 2. Setting Up Jackson & Gson Dependencies

Add Jackson Databind or Gson dependencies to your `pom.xml`:

```xml
<dependencies>
    <!-- Jackson Databind for REST-Assured -->
    <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-databind</artifactId>
        <version>2.16.1</version>
    </dependency>
    <!-- Gson Library -->
    <dependency>
        <groupId>com.google.code.gson</groupId>
        <artifactId>gson</artifactId>
        <version>2.10.1</version>
    </dependency>
</dependencies>
```

---

## 3. Creating Java POJO Classes

Define clean, encapsulated POJOs with Jackson annotations:

**model/UserPayload.java**

```java
package com.mycodeyatra.model;
 
import com.fasterxml.jackson.annotation.JsonIgnoreProperties;
import com.fasterxml.jackson.annotation.JsonProperty;
 
@JsonIgnoreProperties(ignoreUnknown = true)
public class UserPayload {
 
    @JsonProperty("name")
    private String name;
 
    @JsonProperty("job")
    private String job;
 
    public UserPayload() {}
 
    public UserPayload(String name, String job) {
        this.name = name;
        this.job = job;
    }
 
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
 
    public String getJob() { return job; }
    public void setJob(String job) { this.job = job; }
}
```

---

## 4. Executing Automated Tests with Object Mapping

Use REST-Assured to serialize the request POJO and deserialize the response object:

**src/test/java/com/mycodeyatra/JsonParsingTest.java**

```java
package com.mycodeyatra;
 
import com.mycodeyatra.model.UserPayload;
import io.restassured.RestAssured;
import io.restassured.http.ContentType;
import org.junit.jupiter.api.BeforeAll;
import org.junit.jupiter.api.Test;
 
import static io.restassured.RestAssured.given;
import static org.hamcrest.MatcherAssert.assertThat;
import static org.hamcrest.Matchers.*;
 
public class JsonParsingTest {
 
    @BeforeAll
    public static void setup() {
        RestAssured.baseURI = "https://reqres.in/api";
    }
 
    @Test
    public void testSerializationAndDeserialization() {
        // 1. Create Java POJO Request
        UserPayload requestUser = new UserPayload("Pankaj Kumar", "Automation Architect");
 
        // 2. Send POST Request with Object Serialization
        UserPayload responseUser = given()
            .contentType(ContentType.JSON)
            .body(requestUser) // REST-Assured automatically serializes via Jackson
        .when()
            .post("/users")
        .then()
            .statusCode(201)
            .extract()
            .as(UserPayload.class); // Deserializes response JSON into POJO
 
        // 3. Assert values on Java Object
        assertThat(responseUser.getName(), equalTo("Pankaj Kumar"));
        assertThat(responseUser.getJob(), equalTo("Automation Architect"));
    }
}
```

---

## 5. Jackson vs. Gson Comparison

| Feature | Jackson Databind | Gson Library |
| :--- | :--- | :--- |
| **Performance** | High (Optimal for large JSON streams) | Moderate (Great for standard payloads) |
| **Spring Integration** | Default in Spring Boot & REST-Assured | Requires custom configuration |
| **Annotations** | Rich (`@JsonProperty`, `@JsonIgnore`) | Simple (`@SerializedName`) |

---

## Key Best Practices

1. **Use `@JsonIgnoreProperties(ignoreUnknown = true)`**: Prevents tests from breaking when new unmapped fields are added to API responses.
2. **Reuse POJOs Across Tests**: Keep POJO definitions modular in a dedicated `model/` package.
