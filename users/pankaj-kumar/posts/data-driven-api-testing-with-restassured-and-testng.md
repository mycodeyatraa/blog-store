---
title: Data-Driven API Testing with RestAssured and TestNG
date: 11-Feb-2025
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [restassured, java, data-driven, testng, automation]
category: API Testing
categories: [API Testing, Automation, Java]
excerpt: >-
  Master Data-Driven Testing (DDT) by integrating TestNG DataProviders into your RestAssured framework to execute massive datasets through a single automated script.
readTime: 6 min read
---

One of the most crucial concepts in API testing is **Data-Driven Testing (DDT)**. Instead of writing five separate tests to validate how an API handles five different scenarios, you write *one* test and feed it a matrix of data.

When using RestAssured alongside TestNG, you can perfectly leverage TestNG's built-in `@DataProvider` feature to achieve this flawlessly!

---

## 1. Creating the DataProvider

A `@DataProvider` is simply a Java method that returns a two-dimensional array of objects (`Object[][]`). Each row in the array represents a single test execution, and the columns represent the parameters passed to the test method.

Let's create a matrix that tests three scenarios for our Mock Server's `/users` creation endpoint:
1. Creating a valid admin user.
2. Creating a valid regular user.
3. Attempting to create a user with a missing email (expecting an error).

```java
// 1. Define the DataProvider
@DataProvider(name = "userDataProvider")
public Object[][] createUserData() {
    return new Object[][] {
        { "John Doe", "john@example.com", "admin", 201 },
        { "Jane Smith", "jane@example.com", "user", 201 },
        { "Missing Email", null, "user", 400 } // Should fail as expected by the API
    };
}
```

## 2. Consuming the DataProvider in RestAssured

Now, we attach this DataProvider to our `@Test` method. The test method must accept parameters that perfectly match the columns in the `Object[][]` array.

```java
// 2. Consume the DataProvider
@Test(dataProvider = "userDataProvider")
public void testCreateUserWithMultipleData(String name, String email, String role, int expectedStatusCode) {
    System.out.println("\\n--- Executing Data-Driven Test for: " + name + " ---");
    Map<String, Object> payload = new HashMap<>();
    payload.put("name", name);
    if (email != null) {
        payload.put("email", email);
    }
    payload.put("role", role);
    Response response = RestAssured.given()
            .contentType(ContentType.JSON)
            .body(payload)
            .when()
            .post("/api/users")
            .then()
            .extract().response();
    // Validate that the status code matches the expected result from the DataProvider
    Assert.assertEquals(response.getStatusCode(), expectedStatusCode);
    if (expectedStatusCode == 201) {
        System.out.println("Successfully Created User ID: " + response.jsonPath().getString("id"));
        Assert.assertEquals(response.jsonPath().getString("name"), name);
    } else {
        System.out.println("Expected Error Triggered: " + response.jsonPath().getString("error"));
        Assert.assertTrue(response.jsonPath().getString("error").contains("required"));
    }
}
```

## The Execution Output

When we run this suite (`mvn clean test`), TestNG automatically executes the method three separate times!

```text
--- Executing Data-Driven Test for: John Doe ---
Successfully Created User ID: 3e8a9f...
--- Executing Data-Driven Test for: Jane Smith ---
Successfully Created User ID: 489d82...
--- Executing Data-Driven Test for: Missing Email ---
Expected Error Triggered: Name and Email are required
[INFO] Tests run: 3, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 14.21 s -- in TestSuite
```

By decoupling your test data from your test logic, your RestAssured frameworks become incredibly robust, maintainable, and infinitely scalable! You can even connect your DataProviders directly to external Excel or CSV files.

In our final set of tutorials, we will explore Mocking, WireMock, and integrating this framework into Jenkins!
