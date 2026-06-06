---
title: JSON Schema Validation in RestAssured
date: 13-Feb-2025
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [restassured, java, json-schema, validation, testing]
category: API Testing
categories: [API Testing, Automation, Java]
excerpt: >-
  Replace dozens of fragile assertions by learning how to validate entire API payload structures and data types using RestAssured's JSON Schema Validator.
readTime: 6 min read
---

Instead of writing dozens of assertions to check every single property in a JSON response (e.g. `assertEquals`, `assertNotNull`), you can validate the entire structure, data types, and required fields in one single line of code!

This is achieved using **JSON Schema Validation**. 

A JSON Schema is a declarative contract that defines what a valid JSON response should look like. In this tutorial, we will learn how to integrate the `json-schema-validator` module with RestAssured.

---

## 1. Adding the Dependency

First, add the `json-schema-validator` dependency to your `pom.xml`. This module is officially provided by the RestAssured team.

```xml
<dependency>
    <groupId>io.rest-assured</groupId>
    <artifactId>json-schema-validator</artifactId>
    <version>5.5.0</version>
</dependency>
```

## 2. Creating the Schema File

Let's test our mock API's `/health` endpoint. This endpoint returns a simple payload:
```json
{
  "status": "OK",
  "message": "Mock Server is running"
}
```

We need to create a JSON schema that dictates:
1. It must be an `object`.
2. It must have a `status` field of type `string`.
3. It must have a `message` field of type `string`.
4. Both fields are `required`.

Create a file named `schema.json` in your `src/test/resources/` directory:

```json
{
  "$schema": "http://json-schema.org/draft-04/schema#",
  "type": "object",
  "properties": {
    "message": {
      "type": "string"
    },
    "status": {
      "type": "string"
    }
  },
  "required": [
    "message",
    "status"
  ]
}
```

*(Tip: You can use free online tools to automatically generate a JSON Schema from a sample JSON payload!)*

## 3. Validating the Schema in RestAssured

Now, we write the test. We simply import the `matchesJsonSchemaInClasspath` static method and apply it directly inside our `.then().body()` block!

```java
package com.mycodeyatra.tests;
import io.restassured.RestAssured;
import org.testng.annotations.BeforeClass;
import org.testng.annotations.Test;
import static io.restassured.RestAssured.given;
import static io.restassured.module.jsv.JsonSchemaValidator.matchesJsonSchemaInClasspath;
public class JsonSchemaTest {
    @BeforeClass
    public void setup() {
        RestAssured.baseURI = "http://localhost:8080";
        RestAssured.basePath = "/api";
    }
    @Test
    public void testJsonSchemaValidation() {
        System.out.println("\\n--- Executing JSON Schema Validation Test ---");
        given()
            .when()
            .get("/health") 
            .then()
            .statusCode(200)
            // Validating the entire JSON structure perfectly matches schema.json
            .body(matchesJsonSchemaInClasspath("schema.json"));
        System.out.println("JSON Schema validation passed successfully!");
    }
}
```

## The Execution Output

When you run this test (`mvn clean test`), RestAssured parses the network payload against the schema definitions:

```text
--- Executing JSON Schema Validation Test ---
JSON Schema validation passed successfully!
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 13.88 s -- in TestSuite
```

If the API suddenly removed the `status` field or changed it to an integer, this test would instantly fail with a highly descriptive error highlighting the exact schema violation.

JSON Schema validation is an incredibly powerful tool for catching hidden contract breakages! In our next tutorial, we'll dive into advanced Mocking strategies!
