---
title: Setting up RestAssured with Java
date: 02-Feb-2025
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [restassured, java, automation, api-testing]
category: API Testing
categories: [API Testing, Automation, Java]
excerpt: >-
  Now that we understand how REST APIs work, it's time to build our automation framework using Java 17, RestAssured, and TestNG.
readTime: 5 min read
---

Now that we understand how REST APIs work, it's time to build our automation framework! We will be using **Java 17** along with **RestAssured**, the industry standard library for testing and validating REST services in Java.

We will also use **TestNG** as our testing framework to run the code, and we'll configure **Allure** to generate beautiful, visual reports later.

---

## Step 1: Configuring Maven (pom.xml)

We use Maven to handle all of our library dependencies. In your project's pom.xml, we need to add the RestAssured and TestNG libraries. 

Here is the exact dependency configuration we are using for this series:

```xml
<dependencies>
    <!-- RestAssured -->
    <dependency>
        <groupId>io.rest-assured</groupId>
        <artifactId>rest-assured</artifactId>
        <version>5.5.0</version>
    </dependency>
    <!-- JSON Parsing -->
    <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-databind</artifactId>
        <version>2.17.2</version>
    </dependency>
    <!-- TestNG -->
    <dependency>
        <groupId>org.testng</groupId>
        <artifactId>testng</artifactId>
        <version>7.10.2</version>
        <scope>test</scope>
    </dependency>
</dependencies>
```

---

## Step 2: Writing Our First Test

With our dependencies installed, let's write a simple Health Check test. We want to ping our local MyCodeYatra Mock API Server running on port 8080 to ensure it is alive.

Create a new Java class named FirstRestAssuredTest.java in your src/test/java/com/mycodeyatra/tests directory:

```java
package com.mycodeyatra.tests;
import io.restassured.RestAssured;
import io.restassured.response.Response;
import org.testng.Assert;
import org.testng.annotations.BeforeClass;
import org.testng.annotations.Test;
public class FirstRestAssuredTest {
    @BeforeClass
    public void setup() {
        // Set the global base URL for all RestAssured requests
        RestAssured.baseURI = "http://localhost:8080";
        RestAssured.basePath = "/api";
    }
    @Test
    public void testGetUsers() {
        System.out.println("--- Executing GET /api/users ---");
        // Construct and execute the request
        Response response = RestAssured.given()
                .when()
                .get("/users")
                .then()
                .extract().response();
        // Print the result
        System.out.println("Response Status Code: " + response.getStatusCode());
        // Validate the response using TestNG Assertions
        Assert.assertEquals(response.getStatusCode(), 200, "Expected status code 200");
        Assert.assertTrue(response.getBody().asString().contains("data"), "Response body does not contain 'data' array");
        System.out.println("Successfully validated GET /users response!");
    }
}
```

---

## Step 3: Executing the Test

To run the test, we execute the mvn clean test command in our terminal. When we run this against our live mock server, here is the exact console output we receive:

```text
[INFO] Running TestSuite
--- Executing GET /api/users ---
Response Status Code: 200
Successfully validated GET /users response!
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 9.735 s -- in TestSuite
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
```

The server successfully returned a 200 OK status, and our RestAssured script successfully caught and validated it! 

In the next tutorial, we will dive deeper into advanced GET and POST requests, manipulating real data in the mock database!
