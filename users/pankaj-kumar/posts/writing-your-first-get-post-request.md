---
title: Writing your First GET/POST Request
date: 03-Feb-2025
author: pankaj-kumar
authorName: Pankaj Kumar
authorRole: Automation Architect
authorAvatar: https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/pankaj-kumar/images/pankaj.JPG
authorBio: Automation Architect
authorGithub: https://github.com/pankajhyd
authorLinkedin: https://www.linkedin.com/in/pankaj-kumar-94a2b227/
tags: [restassured, java, automation, get, post]
category: API Testing
categories: [API Testing, Automation, Java]
excerpt: >-
  In this tutorial, we will use RestAssured to extract information from our MyCodeYatra Mock API Server using a GET request, and then we will create entirely new data using a POST request.
readTime: 5 min read
---

Now that your Java environment is fully configured, it is time to write real automation tests! In this tutorial, we will use RestAssured to extract information from our MyCodeYatra Mock API Server using a GET request, and then we will create entirely new data using a POST request.

---

## 1. Fetching Data with GET

Our mock server contains a database of 50 randomly generated users. Let's write a RestAssured test to fetch those users, assert that the request was successful (Status 200), and extract the total user count.

Here is the exact code for our test:

```java
@Test(priority = 1)
public void testGetRequest() {
    System.out.println("\n--- Executing GET /users ---");
    Response response = RestAssured.given()
            .when()
            .get("/users")
            .then()
            .extract().response();
    System.out.println("Status Code: " + response.getStatusCode());
    Assert.assertEquals(response.getStatusCode(), 200);
    // Extracting total number of users from JSON
    int totalUsers = response.jsonPath().getInt("total");
    System.out.println("Total Users found: " + totalUsers);
}
```

## 2. Creating Data with POST

Fetching data is easy, but modern automation requires us to actively manipulate the database. To create a new user, we must construct a JSON payload. Instead of manipulating ugly strings, we will use a Java Map which RestAssured will automatically serialize into JSON!

```java
@Test(priority = 2)
public void testPostRequest() {
    System.out.println("\n--- Executing POST /users ---");
    // Creating a JSON payload using a Map
    Map<String, Object> payload = new HashMap<>();
    payload.put("name", "RestAssured Tester");
    payload.put("email", "ra.tester@example.com");
    payload.put("role", "admin");
    Response response = RestAssured.given()
            .contentType(ContentType.JSON) // Tell the server we are sending JSON
            .body(payload)
            .when()
            .post("/users")
            .then()
            .extract().response();
    System.out.println("Status Code: " + response.getStatusCode());
    System.out.println("Response Body: " + response.getBody().asPrettyString());
    Assert.assertEquals(response.getStatusCode(), 201); // 201 Created
    // Validating the inserted data
    String returnedName = response.jsonPath().getString("name");
    Assert.assertEquals(returnedName, "RestAssured Tester");
}
```

---

## The Execution Output

When we execute these two tests in our suite, TestNG runs them in priority order. Here is the exact console output showing a flawless execution against our Mock API Server:

```text
--- Executing GET /users ---
Status Code: 200
Total Users found: 50
--- Executing POST /users ---
Status Code: 201
Response Body: {
    "id": "4a9b269f-ca6e-4356-8dc0-fa40f646dd34",
    "name": "RestAssured Tester",
    "email": "ra.tester@example.com",
    "role": "admin",
    "createdAt": "2026-06-06T18:40:05.649Z"
}
[INFO] Tests run: 2, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 10.84 s -- in TestSuite
```

As you can see, the GET request correctly identified 50 users, and the POST request successfully created our new admin user, instantly returning the newly generated id and createdAt timestamp!

In the next tutorial, we will look at how to dynamically filter this data using Query Parameters and Path Variables.
